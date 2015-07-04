# Promises

**Promises** are an interesting and alternative way to create futures. A Promise can be seen as a *writable, single-assignment* container, which completes a Future.

In Scala, a Promise can be represented by the following trait:

```
trait Promise[T] {
  def future: Future[T]
  def complete(result: Try[T]): Unit
  def tryComplete(result: Try[T]): Boolean
}

trait Future[T] {
  def onCompleted(f: Try[T] => Unit): Unit
}
```

A promise `p` can alternatively complete or fail the future returned by `p.future`. In other words, it acts as a **mailbox** for the future that it wraps.

The Scala standard library already implements Promise in the `scala.concurrent` package.

Promises have a *single-assignment* semantics, so they can be completed only once and usually are used to implement other operator on futures.
