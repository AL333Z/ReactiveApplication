# Implementation on Akka streams

Using Akka Streams abstractions, building a processing pipeline that solves the problem introduced by the requirements is a pretty straight-forward process.

In Akka Streams, to build a processing pipeline the first thing to get is a `Source`. In this case, let's start with an utility class that takes care of interacting to Twitter' APIs, exposing a method `listenAndStream` that returns a `Source[Tweet, Unit]`.

```
class TwitterStreamListener(filter: Array[String], config: Configuration) {

  // ...

  def listenAndStream: Source[Tweet, Unit] = { ... }
}
```

This method will introduce tweets regarding our two brands as they are pubblished.

Once we get the source of our items, two further steps to performs are the ones that assign to each tweet a "static score", and then a valutation.

```
  val g = FlowGraph.closed() { implicit builder: FlowGraph.Builder[Unit] =>
    import akka.stream.scaladsl.FlowGraph.Implicits._

    // our targets brands
    val filters: scala.collection.immutable.List[String]
        = "google" :: "apple" :: List()
    val twitterStreamListener =
        new TwitterStreamListener(filters.toArray, twConfigBuilder.build())

    val in: Source[Tweet, Unit] = twitterStreamListener.listenAndStream

    // assign a "static" score
    val mapToScore: Flow[Tweet, (Tweet, Double), Unit]
        = Flow[Tweet].map(tweet => (tweet,
      (1.0 + 1.01 * tweet.author.followerCount + 1.02 * tweet.retweetCount)))

    // extract the "sentiment" from the tweet (not implemented for real..)
    val mapToValutation: Flow[(Tweet, Double), (Tweet, Double), Unit]
        = Flow[(Tweet, Double)].map(tweetScoreTuple =>
            if ( /*... positive or negative content? ...*/)
                (tweetScoreTuple._1, tweetScoreTuple._2)
            else (tweetScoreTuple._1, -tweetScoreTuple._2)
    )

    // build the pipeline
    in ~> mapToScore ~> mapToValutation
  }
```

At this point, the last node of the linear graph has type `Flow[(Tweet, Double), (Tweet, Double), Unit]`, meaning that:
- it accepts a tuple containing a Tweet and a Double, and it puts out the same type;
- looking at the code itself, if returns a stream of pairs of tweets and valutations.

The next step allows to determine in which category (brand) each tweet in the stream belongs to. This step is needed, since the informations that come from the APIs refer to both the brands.

```
// for each tweet, detect in which category the tweet has been returned,
// and return a tuple with this additional information (so we can now group
// the tweet for category)
val mapWithCategory: Flow[(Tweet, Double), (String, Double), Unit]
    = Flow[(Tweet, Double)].mapConcat(tuple =>
  filters
    .filter(f => tuple._1.body.toLowerCase.contains(f.toLowerCase))
    .map(matchedFilter => (matchedFilter, tuple._2)))

// build the pipeline
in ~> mapToScore ~> mapToValutation ~> mapWithCategory
```

Note that now the stream puts out a tuple of `String` and `Double`, that correspond to the current category and valutation for the `Tweet` arrived from the input. At this stage, the other further information about the tweet are no more necesary.

At this point, the stream of tuples needs to grouped for brand, and then reduced by applying a function that aggregates all the partial result to compose a result. In this simplified case, this function is a sum.

```
// ...

// groups the tweets for brand and sum the valutations
def sumReducedByKey: Flow[(String, Double), (String, Double), Unit]
    = reduceByKey(
  filters.length,
    groupKey = (elem: (String, Double)) => elem._1,
    foldZero = (key: String) => (0.0))(fold
        = (count: Double, elem: (String, Double)) => elem._2 + count)

  // generic reduce by key function
  def reduceByKey[In, K, Out](maximumGroupSize: Int,
    groupKey: (In) => K,
    foldZero: (K) => Out)(fold: (Out, In) => Out): Flow[In, (K, Out), Unit] = {

    val groupStreams = Flow[In].groupBy(groupKey)
    val reducedValues = groupStreams.map {
      case (key, groupStream) =>
        groupStream.runFold((key, foldZero(key))) {
          case ((key, aggregated), elem) =>
            val newAggregated = fold(aggregated, elem)
            // println("Folding: key: " + key + " aggregate: " + newAggregated)
            (key, newAggregated)
        }
    }

    reducedValues
        .buffer(maximumGroupSize, OverflowStrategy.fail)
        .mapAsyncUnordered(4)(identity)
}

// build the pipeline
in ~> mapToScore ~> mapToValutation ~> mapWithCategory ~> sumReducedByKey ~> out
```

This step of the pipeline is the most difficult to understand at first sight.

By mapping over the groups that contains only the data for a single brand, and using `runFold` (that automatically materializes and runs each sub-stream it is used on) what is returned is a stream with elements of `Future[(String, Double)]`.
Finally, this stream is flattened by calling `mapAsyncUnordered(4)(identity)`, that gets the values out of the futures and then injects these values in the returned stream.

One last thing to note is the presence of a buffer between the mapAsyncUnordered and the actual stream of futures.

From the documentation, where the reduceByKey approach has been taken:

>The reason for this is that the substreams produced by groupBy() can only complete when the original upstream source completes. This means that mapAsync() cannot pull for more substreams because it still waits on folding futures to finish, but these futures never finish if the additional group streams are not consumed. This typical deadlock situation is resolved by this buffer which either able to contain all the group streams (which ensures that they are already running and folding) or fails with an explicit failure instead of a silent deadlock.

Putting all the pieces together, and adding a limit to 1000 tweets for the sake of brevity, the overall code looks like the following.

```
object MainStreamingExample extends App {

  // ... config twitter APIs and client ...

  // ActorSystem & thread pools
  val execService: ExecutorService = Executors.newCachedThreadPool()
  implicit val system: ActorSystem = ActorSystem("ciaky")
  implicit val ec: ExecutionContext = ExecutionContext.fromExecutorService(execService)
  implicit val materializer = ActorMaterializer()

  val g = FlowGraph.closed() {
    implicit builder: FlowGraph.Builder[Unit] =>

    import akka.stream.scaladsl.FlowGraph.Implicits._

    // our targets brands
    val filters: scala.collection.immutable.List[String] = "google" :: "apple" :: List()
    val twitterStreamListener = new TwitterStreamListener(filters.toArray, twConfigBuilder.build())

    val in: Source[Tweet, Unit] = twitterStreamListener.listenAndStream
    val out: Sink[(String, Double), Future[Unit]]
        = Sink.foreach[(String, Double)](t =>
      println("[Out] key: " + t._1 + " partial score: " + t._2))

    // limit the elaboration to 1000 tweets
    val take: Flow[Tweet, Tweet, Unit] = Flow[Tweet].take(1000)

    // assign a "static" score
    val mapToScore: Flow[Tweet, (Tweet, Double), Unit]
        = Flow[Tweet].map(tweet => (tweet,
            (1.0 + 1.01 * tweet.author.followerCount)))

    // extract the "sentiment" from the tweet (not implemented for real..)
    val mapToValutation: Flow[(Tweet, Double), (Tweet, Double), Unit]
        = Flow[(Tweet, Double)].map(tweetScoreTuple =>
            if (/*... positive or negative content? ...*/)
                (tweetScoreTuple._1, tweetScoreTuple._2)
            else (tweetScoreTuple._1, -tweetScoreTuple._2)
    )

    // for each tweet, detect in which category the tweet has been returned, and return a tuple with this
    // additional information (so we can now group the tweet for category)
    val mapWithCategory: Flow[(Tweet, Double), (String, Double), Unit]
        = Flow[(Tweet, Double)].mapConcat(tuple =>
      filters
        .filter(f => tuple._1.body.toLowerCase.contains(f.toLowerCase))
        .map(matchedFilter => (matchedFilter, tuple._2))
    )

    // groups the tweets for category and sum the valutations
    def sumReducedByKey: Flow[(String, Double), (String, Double), Unit]
        = reduceByKey(
            filters.length,
            groupKey = (elem: (String, Double)) => elem._1,
            foldZero = (key: String) => (0.0))(
            fold = (count: Double, elem: (String, Double)) => elem._2 + count)

    // generic reduce by key function
    def reduceByKey[In, K, Out](maximumGroupSize: Int,
      groupKey: (In) => K,
      foldZero: (K) => Out)(fold: (Out, In) => Out): Flow[In, (K, Out), Unit] = {

      val groupStreams = Flow[In].groupBy(groupKey)
      val reducedValues = groupStreams.map {
        case (key, groupStream) =>
          groupStream.runFold((key, foldZero(key))) {
            case ((key, aggregated), elem) =>
              val newAggregated = fold(aggregated, elem)
              println("Folding: key: " + key + " aggregate: " + newAggregated)
              (key, newAggregated)
          }
      }

      reducedValues
        .buffer(maximumGroupSize, OverflowStrategy.fail)
        .mapAsyncUnordered(4)(identity)
    }

    // build the pipeline
    in ~> take ~> mapToScore ~> mapToValutation ~> mapWithCategory ~> sumReducedByKey ~> out
  }

  g.run()
}
```

Even if Akka Streams processing stages are executed concurrently by default, the approach adopted for the `reduceByKey` function leads to a sequential processing stage for the reduction phase, with no parallelization at all.
