# Reactive

Following the idea to provide APIs that starts with basic event handling and ends in an embedded higher-order dataflow language, the framework introduces a generic trait `Reactive`, that factors out the possible concrete abstractions in the following way:

```scala
trait Reactive[+Msg, +Now] {
    def current(dep: Dependant): Now
    def message(dep: Dependant): Option[Msg]
    def now: Now = current(Dependent.Nil)
    def msg: Msg = message(Dependent.Nil)
}
```

The trait `Reactive` is defined based on two type parameters: one for **the message type an instance emits** and one for **the values it holds**.

Starting from the previous base abstraction, two further types can be defined:

```scala
trait Signal[+A] extends Reactive[A,A]
trait Events[+A] extends Reactive[A,Unit]
```

The difference between the two types can be seen directly in the types:
- In `Signal`, `Msg` and `Now` types are identical.
- In `Events`, `Msg` and `Now` types differ. In particular, the type for the type parameter `Now` is `Unit`. This means that for an instance of `Events` the notion of "current value" has no sense at all.

The two subclasses need to implement two methods which obtain the reactive’s current message or value and create dependencies in a single turn.

The next two sections will better examine the two abstraction introduced here.

NB: the examples and code provided in this chapter have been taken directly from the paper itself.

## Events

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

## Signal

The **Signal** type represents the other half of the story.

In programming a large set of problems is about **synchronizing data that changes over time**, and signals are introduced to overcome these needs.

In simple words, a `Signal` is the **continuous** counterpart of trait `Events` and represents **time-varying values**, mantaining:
- its **current value**
- the current **expression** that defines the signal value
- a set of **observers**: the other signals that depend on its value

A concrete type for `Signal` is the `Var`, that abstract the notion of **variable signal** and is defined as follows:

```scala
class Var[A](init: A) extends Signal[A] {
    def update(newValue: A): Unit = ...
}
```

A `Var`'s current value can change when somebody calls an `update( )` operation on a it or the value of a dependant signal changes.

**Constant signals** are represented by `Val`:

```scala
class Val[A](value: A) extends Signal[A]
```

To compose signals, the framework doesn't provide combinator methods at all, but introduce the notion of **signal expressions**, indeed.
To better explain the concept, let's look at an simple example.

```scala
val a = new Var(1)
val b = new Var(2)
val sum = Signal{ a()+b() }

observe(sum) { x => println(x) }

a()= 7
b()= 35
```

Signals are primarily used to create variable dependencies as seen above. In other words, all the "plumbing work" of connecting the dependencies and propagating the changes is already provided by the framework itself.

The framework and the Scala language provide a convenient and simple syntax to get and update the current value of a Signal, and also to create a variable dependencies between signals. For example:
- the code `Signal{ a()+b() }` creates a dependencies that binds the changes from `a` and `b` to be propagated and evaluated in the `sum` signal
- the code `a()= 7` is evaluated as `a.update(7)`
- the framework also provide an implicit converter that enables to create easily `Val` signals
