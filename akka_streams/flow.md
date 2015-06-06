# Flow and RunnableFlow

The previous sections introduced the two ends of a computation. This section will introduce the **Flow** abstraction: a processing stage which has exactly one input and one output. In Scala, the `Flow` type is defined as follows.

```scala
class Flow[In, Out]
```

The `Out` type is the type of the elements the flow returns and the `In` type is the type of the elements the flow accepts.

A **RunnableFlow** is a flow that has both ends attached to a source and sink, and represents *a flow that can run*. A trivial example of `RunnableFlow` in action is the following:

```scala
val in: Source[Tweet] = Source(...)
val out: Sink[Tweet] = Sink.foreach(println)
val runnableFlow: RunnableFlow = in.to(out)

runnableFlow.run()
```

As already introduced, Akka Streams implements an *asynchronous non-blocking back-pressure protocol* standardised by the Reactive Streams specification, and the user doesn't have to manually manage back-pressure handling code manually since this is already provided by the library itself with a default implementation.

In this regards, there are two main scenarios: **slow publisher with fast subscriber** and **fast publisher with slow subscriber**.

The best scenario is the former, where all the items produced by the publisher are always delivered to the subscriber with no delay due to a lack of demand. Infact, the protocol guarantees that the publisher will never signal more items than the signalled demand, and since the subscriber however is currently faster, it will be signalling demand at a higher rate, so the publisher should not ever have to wait in publishing its items. This scenario is also referred as *push mode*, since the publisher is never delayed in pushing items to the subscriber.

The latter case if more problematic, since the subscriber is not able to keep the pace of the publisher in accepting incoming items and needs to signal its lack of demand to the publisher.
Since the protocol guarantees that the publisher will never signal more items than the signalled demand, there the following strategies availlable:

- not generate items, if it can control its production
- buffering the items within a bounded queue, so the items can be provided when more demand is signalled
- drop items until more demand is signalled
- as an ultimate strategy, tear down the stream if none of the previous strategies are applicable

This latter scenario is also referred as *pull mode*, since the subriber drives the flow of items.
