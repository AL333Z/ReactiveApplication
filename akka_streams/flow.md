# Flow and RunnableFlow

The previous sections introduced the two ends of a computation. This section will introduce the **Flow** abstraction: a processing stage which has exactly one input and one output. In Scala, the `Flow` type is defined as follows.

```scala
class Flow[In, Out]
```

The `Out` type is the type of the elements the flow returns and the `In` type is the type of the elements the flow accepts.

A **RunnableFlow** is a flow that has both ends attached to a source and sink, and represents *a flow that can run*.

```scala
val in: Source[Tweet] = Source(...)
val out: Sink[Tweet] = Sink.foreach(println)
val runnableFlow: RunnableFlow = in.to(out)

runnableFlow.run()
```
