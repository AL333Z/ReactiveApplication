# Implementation on RxScala

Also using RxScala (RxJava ported to Scala) abstractions, building a processing pipeline that solves the problem introduced by the requirements is a pretty simple process.

To build a processing pipeline the first thing to get is a `Observable`. Following what it was done for the previous implementation, let's start with an utility class that takes care of interacting to Twitter' APIs, exposing a method listenAndStream that returns an `Observable[Tweet]`.

```scala
class TwitterStreamListener(filter: Array[String], config: Configuration) {

  // ...

  def listenAndStream: Observable[Tweet] = { ... }
}
```

Once we get the source of tweets, two further steps to performs are the ones that assign to each tweet a "static score", and then a valutation.

```scala
val filters: scala.collection.immutable.List[String] = "google" :: "apple" :: List() // our targets brands
val twitterStreamListener = new TwitterStreamListener(filters.toArray, twConfigBuilder.build())

twitterStreamListener.getTweetObservable
    .map(tweet => (tweet, (1.0 + 1.01 * tweet.author.followerCount + 1.02 * tweet.retweetCount)))
    .map(tweetScoreTuple =>
      if (/*... positive or negative content? ...*/) (tweetScoreTuple._1, tweetScoreTuple._2)
      else (tweetScoreTuple._1, -tweetScoreTuple._2)
    )
```

At this point, the resulting observable has type `Observable[(Tweet, Double)]`, meaning that:
- it accepts a tuple containing a `Tweet` and a `Double`, and it puts out the same type;
- looking at the code itself, if returns a stream of pairs of tweets and valutations.

As in the previous implementation, the next step allows to determine in which category (brand) each tweet in the stream belongs to. This step is needed, since the informations that come from the APIs refer to both the brands. To perform the work, a simple flatMap will take care of this job.

```scala
    //...
    .flatMap(tuple => {
      val matchedFilters = filters
        .filter(f => tuple._1.body.toLowerCase.contains(f.toLowerCase))
        .map(matchedFilter => (matchedFilter, tuple._2))
      Observable.from(matchedFilters)
    })
```

Note that, also with this implementation, now the stream puts out a tuple of `String` and `Double`, that correspond to the current category and valutation for the `Tweet` arrived from the input.

At this point, the stream of tuples needs to grouped for brand, and then reduced by applying a function that aggregates all the partial result to compose a result. This step has a really shorter and concise syntax in this RxScala's implementation.

```scala
  val groupKey: ((String, Double)) => String = tuple => tuple._1
  val foldZero: (String) => Double = key => 0.0
  val fold: (Double, (String, Double)) => Double = (count: Double, elem: (String, Double)) => elem._2 + count

    //...
    .groupBy(groupKey)
    .flatMap {
      case (key, groupStream) => groupStream.scan((key, foldZero(key))) {
        case ((key, aggregated), elem) => (key, fold(aggregated, elem))
      }
    }
```

The groupBy operator creates a new observable for each brand. Each sub-observable is then reduced and flattened.

Putting all pieces together, the overall code is the following.

```scala
object MainRxExample extends App {

  // ... config twitter APIs and client ...

  val filters: scala.collection.immutable.List[String] = "google" :: "apple" :: List() // our targets brands
  val twitterStreamListener = new TwitterStreamListener(filters.toArray, twConfigBuilder.build())

  val groupKey: ((String, Double)) => String = tuple => tuple._1
  val foldZero: (String) => Double = key => 0.0
  val fold: (Double, (String, Double)) => Double = (count: Double, elem: (String, Double)) => elem._2 + count

  twitterStreamListener.getTweetObservable

     // assign a "static" score
    .map(tweet => (tweet, (1.0 + 1.01 * tweet.author.followerCount + 1.02 * tweet.retweetCount)))

    // extract the "sentiment" from the tweet
    .map(tweetScoreTuple =>
      if (/*... positive or negative content? ...*/) (tweetScoreTuple._1, tweetScoreTuple._2)
      else (tweetScoreTuple._1, -tweetScoreTuple._2)
    )

    // groups the tweets for brand and sum the valutations
    .flatMap(tuple => {
      val matchedFilters = filters
        .filter(f => tuple._1.body.toLowerCase.contains(f.toLowerCase))
        .map(matchedFilter => (matchedFilter, tuple._2))
      Observable.from(matchedFilters)
    })

    // groups the tweets for brand and sum the valutations
    .groupBy(groupKey)
    .flatMap {
      case (key, groupStream) => groupStream.scan((key, foldZero(key))) {
        case ((key, aggregated), elem) => (key, fold(aggregated, elem))
      }
    }

    .subscribe(t => println("[Out] key: " + t._1 + " partial score: " + t._2))
}
```
