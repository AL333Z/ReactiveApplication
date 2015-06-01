# Source

In Akka Streams, a **Source** is a set of stream processing steps that has **one open output**. In Scala, the `Source` type is defined as follows:

```scala
final class Source[Out]
```

The `Out` type is the type of the elements the source produces.

It either can be an atomic source or it can comprise any number of internal sources and transformation that are wired together. Some examples of the former case is given from the following code, that shows some of the utility constructor for the `Source` type.

```scala
// Create a source from an Iterable
Source(List(1, 2, 3))

// Create a source from a Future
Source(Future.successful("Hello Streams!"))

// Create a source from a single element
Source.single("only one element")

// an empty source
Source.empty
```

An example of the latter case is when a `Flow` is attached  to a `Source`, resulting in a composite source, as in the following example.

```scala
val source: Source[Tweet] = Source(...)
val filter: Flow[Tweet, Tweet] = Flow[Tweet].filter(t => t.hashtags.contains(hashtag))
val compositeSource: Source[Tweet] = tweets.via(filter)
```

The `via( )` method transforms a source by appending the given processing stages, and it's the glue that enables to build composite sources.
