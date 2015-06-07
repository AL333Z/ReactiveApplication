# Events

The first type to take in consideration is the `Event` type.

To simplify the event handling logic in an application, the framework provides a general and uniform event interface, with `EventSource`. An `EventSource` is an entity that can *raise* or *emit* events at any time. For example:

```scala
val es = new EventSource[Int]
es raise 1
es raise 2
```

To attach a side-effect in response to an event, an observer has to *observe* the event source, providing a closure. Continuing with the previous example, the following code prints all events from the event source to the console.

```scala
val ob = observe(es) { x =>
    println("Receiving " + x)
}
...
ob.dispose()
```

`observe( )` returns an handle of the observer,that can be used to uninstall and dispose the observer prematurely, via its `dispose()` method.
This is a common pattern in all of the other frameworks/libraries presentend in this thesis.

The basic types for events handling are pretty neat and simple to reason about, since they are first-class values. The usage of these types starts to be helpful only if combined with a set of operators, that enables developers to build better and declarative abstration.

For example, the `Events` trait defines some common operators as follows:

```scala
def merge[B>:A](that: Events[B]): Events[B]
def map[B](f: A => B): Events[B]
def collect[B](p: PartialFunction[A, B]): Events[B]
```

When building abstractions, the developer doesn't need to take care of the events propagation, since this's provided by the framework itself.
