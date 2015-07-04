# Reactive Streams

>Reactive Streams is an initiative to provide a standard for asynchronous stream processing with non-blocking back pressure. This encompasses efforts aimed at runtime environments (JVM and JavaScript) as well as network protocols.

The project is a collaboration between engineers from Kaazing, Netflix, Pivotal, RedHat, Twitter, Typesafe and many others and is developed and discussed in the open.

The core idea behind the Reactive Stream standard is in the definition of two different channels for the **downstream data** and the **upstream demand**.

![](https://github.com/pkinsky/akka-streams-example/blob/master/img/stream.png?raw=true)

This allows to overcome one of the biggest issue of the Rx approaches: **backpressure**. Backpressure is a lack of demand, and is due to the fact that the producer of a stream of items is faster than the consumer, and that difference of speed quickly determines a growing backlog of unconsumed items on the consumer side.

In the literature (and in our everyday life) there's already a protocol that solved a similar problem: TCP.
With this in mind, engineers behind Reactive Streams proposed a solution that, in simple terms, works like this:

- A publisher doesn't send data until a request arrives via the demand channel, at which point it can push a certain number of elements (in according to the request) downstream.
- When outstanding demand exists, the publisher is free to push data to the subscriber.
- When demand is exhausted, the publisher cannot send data except as a response to demand signalled from downstream.

Engineers called this technique **dynamic push/pull**. The dynamic terms indicates that the system should **adapt** to the current conditions of its components and that's not safe to only use a *just push* or just *pull approach*.

A **just push** approach is not safe when the **subscriber is slow**, since it'll quickly start to be overwhelmed by the offers and will start to:

- drop items, in the case of it use a bounded buffer to store received messages (just like TCP)
- trigger an out of memory error

A **just pull** approach is too slow in the case that the **subscriber is faster** than the pubblisher.

Two important notes about the fact that data and demand *flows* in different channels and directions are the following:

- **Merging streams splits the upstream demand**

![Merging streams splits the upstream demand](https://github.com/pkinsky/akka-streams-example/blob/master/img/merge.png?raw=true)

- **Splitting streams merges the downstream demand**
![Splitting streams merges the downstream demand](https://github.com/pkinsky/akka-streams-example/blob/master/img/split.png?raw=true)

As the main document says, the scope of Reactive Streams is to find a minimal set of interfaces, methods and protocols that will describe the necessary operations and entities to achieve the goal—asynchronous streams of data with non-blocking back pressure. This set of interfaces are a low level specification to which each library should be conform to.

The API offers the following interfaces that are required to be implemented by each implementations:

- Subscriber
- Subscription
- Publisher
- Processor
