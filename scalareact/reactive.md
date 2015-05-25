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
