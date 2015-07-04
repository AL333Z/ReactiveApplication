# Futures

A Future is an immutable handle to a value or a failure that will become availlable in the future time. Another possible definition is that Futures are monads that handles **exceptions** and **latency**. A fist simple possible definition in Scala is:

```
trait Future[T] {
    def onComplete(callback: Try[T] => Unit)
        (implicit executor: ExecutionContext): Unit
}

object Future {
    def apply(body: => T)
        (implicit context: ExecutionContext): Future[T]
}
```

This first definition put some important concepts directly in the type definitions:

- a Future is parametrized on a type `T`
- a Future may **fail**, returning a `Try.Failure`
- a Future may **succeed** (successfully completed), returning a `Try.Success[T]`

Futures are really useful in defining operations that will be executed at some point in time by some thread (execution context). Futures are immutable from the developer point of view: once an API returns a Future, there's is no way to set a value insede a Future. In other words, a Future may only be assigned once.

A more complete and precise definition for Future is the following:

```
trait Awaitable[T] extends AnyRef {
    abstract def ready(atMost: Duration): Unit
    abstract def result(atMost: Duration): T
}

￼￼trait Future[T] extends Awaitable[T] {
    def filter(p: T => Boolean): Future[T]
    def flatMap[S](f: T => Future[S]): Future[U]
    def map[S](f: T => S): Future[S]
    def recoverWith(f: PartialFunction[Throwable, Future[T]]): Future[T]
}

object Future {
    def apply[T](body : => T): Future[T]
}
```

In the scala library, Futures are already implemented in the `scala.concurrent` package.
In this package, `Future[T]` is a type which denotes future objects, whereas `future` is a method which creates and schedules an asynchronous computation, and then returns a future object which will be completed with the result of that computation. An example of the usage of Future coming from the Scala documentation is the following:

```
import scala.concurrent._
import ExecutionContext.Implicits.global

val session = socialNetwork.createSessionFor("user", credentials)
val f: Future[List[Friend]] = future {
  session.getFriends()
}
```

The the previous example shows some key points:

-  To obtain the list of friends of a user, a request has to be sent over a network, which can take a long time. Wrapping the request inside the `future` method **doesn't block** the rest of the program execution while **waiting** for a response.
- All the computation is performed **asynchronously**.
- The list of friends becomes available in the future once the server responds.
- If the request fails, the future itself will fail.

To obtain the value of a Future, the library offers two main solution:

- the client **blocks** its computation and wait until the future is completed
- the client **register a callback** by using the `onComplete` method, that was alredy depicted at the beginning of this chapter.

The library also provide the companion methods `onSuccess` and `onFailure`, that usually are used alternatively to `onComplete`. All these three methods have `Unit` as the result type. This is intentional, since the invocations of these methods cannot be chained.

The presence of these callbacks may lead to what is known as "asynchronous spaghetti", where the overall computation is fragmented into asynchronous handlers. To prevent this possibility, futures provide combinators which allow a more straightforward composition, such as `map`, `flatMap`, `filter`, ...

The `map` methods takes a future and a mapping function for the value of the future and produces a new future that is completed with the mapped value once the original future is successfully completed.

The `flatMap` method takes a function that maps the value to a **new future** g, and then returns a future which is completed once g is completed.

The `filter` method creates a new future which contains the value of the original future only if it satisfies some predicate. Otherwise, the new future is failed with a `NoSuchElementException`.

The presence of these methods enable the Future type to also supports for-comprehension. An example, taken from the documentation, of for-comprehension in practice is the following:

```
val usdQuote = future { connection.getCurrentValue(USD) }
val chfQuote = future { connection.getCurrentValue(CHF) }

val purchase: Future[Int] = for {
  usd <- usdQuote
  chf <- chfQuote
  if isProfitable(usd, chf)
} yield connection.buy(amount, chf)

purchase onSuccess {
  case _ => println("Purchased " + amount + " CHF")
}
```

that is translated to:

```
val purchase: Future[Int] = usdQuote flatMap {
  usd =>
  chfQuote
    .withFilter(chf => isProfitable(usd, chf))
    .map(chf => connection.buy(amount, chf))
}
```

For-comprehension is a really elegant and concise way to express **chain of computations**, leveraging the monadic nature of Futures to describe the "**happy path**" of the chain and to propagate (**materialize**) the exceptions.

