# Sink

The dual to the `Source` type is the **Sink** type, that abstracts a set of stream processing steps that has one open input and an attached output. In Scala the `Sink` type is defined as follows:

```scala
final class Sink[In]
```

The `In` type is the type of the elements the sink accepts.

It either can be an atomic sink or it can comprise any number of internal sinks and transformation that are wired together. Some examples of the former case is given from the following code, that shows some of the utility constructor for the `Sink` type.

```scala
// Sink that folds over the stream and returns a Future
// of the final result as its materialized value
Sink.fold[Int, Int](0)(_ + _)

// Sink that returns a Future as its materialized value,
// containing the first element of the stream
Sink.head

// A Sink that consumes a stream without doing anything with the elements
Sink.ignore

// A Sink that executes a side-effecting call for every element of the stream
Sink.foreach[String](println(_))
```

An example of the latter case is given by a Flow that is prepend to a Sink to get a new composite sink, as in the following example:

```scala
val sum: Flow[(Long, Tweet), (Long, Tweet)] = Flow[(Long, Tweet)].scan[(Long, Tweet)](0L, EmptyTweet)(
      (state, newValue) => (state._1 + 1L, newValue._2))
val out: Sink[(Long, Tweet)] = Sink.foreach[(Long, Tweet)]({
      case (count, tweet) => println(count + " white&gold(s). Current tweet: " + tweet.body + " -  " + tweet.author.handle)
    })

val compositeOut: Sink[(Long, Tweet)] = sum.to(out)
```
