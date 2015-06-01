# RxJava

The following table represents a possible classification of the **effects** in programming.

| | One | Many |
| -- | -- | -- |
| Synchronous | T/Try[T] | Iterable[T] |
| Asynchronous | Future[T] | **Observable[T]** |

The appendix on **futures and promises** covers the case of a computation that returns a **single value**. This chapter will focus on the abstraction of **Observables**, analyzing RxJava as a case of study.

As the title of the repository states, *RxJava is a library for composing asynchronous and event-based programs using observable sequences for the JVM*.

The library was heavily inspired by Rx.NET by Microsoft and is developped by Netflix and other contributors. The code is open source and recently, after 2 years of development, has reached the 1.0 version.

RxJava is also conform to the Reactive Streams initiative (see the appendix), and this means that the hard problem of propagating and reacting to back-pressure has been incorporated in the design of RxJava already, and also that it interoperate seamlessly with all other Reactive Streams implementations.

All the theory and the development of the reactive extensions libraries started with an intuition of Erik Meijer, that theorized that **enumerable/enumerator** are **dual** to **observer/observable**.
And this hypothesis is what let us to relate all the principal effects in programming, where in one axis there's is the nature of the computation (sync or async) and in the other one there's the cardinality of the result (one or many).
