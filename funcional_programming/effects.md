# Effects

A possible classification of the essential effects in programming is given by the following table from Erik Meijer.

| | One | Many |
| -- | -- | -- |
| Synchronous | T/Try[T] | Iterable[T] |
| Asynchronous | Future[T] | Observable[T] |

The previous section introduced the concept of monad as a type that allows chainability and containment of computations and introduced the Try type as a monad that handles exceptions.

The Try type is an example of **effect of computation** and put the fact that a computation can fail explicitly in the type signature. The Try type automate the handling of failures, leading to the *happy path* when exceptions don't happen and taking care of the propagation of the error, breaking tha chain, when an error happens.

The second row of the table consider effects that derive from asynchronous computations. In particular, the effect in this case is **latency**, that is a really important argument in reactive programming.

##Latency as an effect

Also latency can be managed and kept under control by using proper abstractions. Two powerful abstaction are `Future` and `Observable`.

Future is a monad that hanldes exceptions and **latency**, and helps in binding the domain login to the *happy path* of the computation.

Futures allows the developer to wrap long running computations in background threads with a functional interface (and operators, such as flatMap, ..).

A possible trait for Future is the following.

```scala
trait Future[T] {
    def onComplete(callback: Try[T] => Unit): Unit
}
```

The fact that the callback receives a Try is really meaningful, since it keeps explicit the fact that the computation can fail. The power of futures is all about having an asynchronous computation wrapped in a functional abstraction.

The generalization of a future that asynchrounosly returns many values is called **Observable**. A possible trait for Observable is the following.

```scala
trait Observable[T] {
   def Subscribe(Observer[T] observer): Subscription
}

trait Observer[T] {
   def onNext(T value): Unit
   def onError(Throwable error): Unit
   def onCompleted(): Unit
}

trait Subscription {
   def unsubscribe(): Unit
}
```

Futures and observables will be covered in depth later in this thesis.
