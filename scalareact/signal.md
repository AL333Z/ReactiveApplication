# Signal

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

