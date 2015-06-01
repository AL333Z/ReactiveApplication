# Akka Streams

**Akka Streams** is a library that is developed on top akka actors, and aims to provide a better tool for building *ephemeral transformation pipelines*.

Akka actors are used as a building block to build an higher abstraction. Some of the biggest issues on building systems on untyped actor are the following:
- actors does **not compose** well
- actors are **not** completely **type safe**
- dealing with an high number of actors, with also a complex logic behind each behaviors, is really error prone and bring back an evil concept: global **state**.

An actor can be seen as a single *unit of consistency*.

From the introduction of the documentation of akka streams:

> Actors can be seen as dealing with streams as well: they send and receive series of messages in order to transfer knowledge (or data) from one place to another. We have found it tedious and error-prone to implement all the proper measures in order to achieve stable streaming between actors, since in addition to sending and receiving we also need to take care to not overflow any buffers or mailboxes in the process. Another pitfall is that Actor messages can be lost and must be retransmitted in that case lest the stream have holes on the receiving side. When dealing with streams of elements of a fixed given type, Actors also do not currently offer good static guarantees that no wiring errors are made: type-safety could be improved in this case.

To overcome these issues, the developers behind Akka started developing Akka Streams, a set of APIs that offers an intuitive and safe way to build **stream processing pipelines**, with a particular attention to efficiency and bounded resource usage.

Akka Streams is also conform to the Reactive Streams initiative (see the appendix), and this means that the hard problem of propagating and reacting to back-pressure has been incorporated in the design of Akka Streams already,  and also that Akka Streams interoperate seamlessly with all other Reactive Streams implementations.

In Akka Streams, a linear processing pipeline can be expressed using the following building blocks:
- **Source**: A processing stage with exactly one output, emitting data elements whenever downstream processing stages are ready to receive them, respecting their demand.
- **Sink**: A processing stage with exactly one input, signalling demand to the upstream and accepting data elements in response, as soon as thei're produced.
- **Flow**: A processing stage which has exactly one input and output, which connects its upstream and downstreams by transforming the data elements flowing through it.
- **RunnableFlow**: A Flow that has both ends attached to a Source and Sink respectively, and is ready to be run().

The APIs also offer another core abstraction to build computation graphs:
- **Graphs**: A processing stage with multiple flows connected at a signle point.

The next sections will depict all the core abstractions introduced here.
