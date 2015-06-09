# Event and Signal

The first abstraction that the framework introduces is the notion of event. An **event** enables to account for *discrete phenomena*, and each of which has a stream (finite or infinite) of occurrences.* Each occurrence is a value paired with a time*. Events are considered to be improving list of occurences.
Or, in simpler words, *events are things that happens*.

In RAC, events are first-class citizens, with the following type:

```swift
public enum Event<T, E: ErrorType> {
	case Next(Box<T>)
	case Error(Box<E>)
	case Completed
	case Interrupted
}
```

The `Event` type, taken alone, doesn't say much. It's just an enum with some possible related values.

>In Swift, an enum is a value type that defines a common type for a group of related values and enables you to work with those values in a type-safe way within your code.

The other foundamental type in RAC is the `Signal` type, defined as:

```swift
public final class Signal<T, E: ErrorType>
```

A signal is just a sequence of events in time that is conforms to the grammar `Next* (Error | Completed | Interrupted)?`.
The grammar introduces a precise semantics and clearify the meaning of the concrete possible instances for the Event type:
- `.Next` represents an event that carry information of a given type `T`
- `.Error` represents an event that carry an error of a given type `E`
- `.Completed` represents a successful terminal event
- `.Interrupted` represents an event that indicates that the signal has been interrupted

Just like RxJava's Observable, also RAC's signal can be distincted in two main categories: hot and cold signals. The main difference from RxJava's implementation is that in RAC this foundamental distinction can be found directly in the types. Infact, RAC defines `Signal`as hot and `SignalProducer` as cold signals.

*NB*: the previous introduction of Signal is general and applicable on both Signal and SignalProducer.

The next sections will go deeper and explore more on Signals and SignalProducers.

## Signal

As introduced previously, Signal implements the abstraction of **hot signal** as a signal that typically has **no start and no end**.

A typical pratical example for this type can be represented by the press events of a button or the arrive of some notifications. They are just events that happens with no relevance of the fact that someone is observing them.

Using a popular philosophical metaphore:

>If a tree falls in a forest and no one is around to hear it, **it does make a sound**.

Looking at the documentation, a Signal is defined as a *push-driven stream that sends Events over time*. Events will be sent to all observers at the same time. So, if a Signal has different observers subscribed to its events, every observer will always see the same sequence of events.

The documentation says another crucial thing, that clarifies the modelling choice made by the developers:

> Signals are generally used to represent event streams that are already "in progress", like notifications, user input, etc. To represent streams that must first be _started_, see the SignalProducer type.

The RAC' mantainers decided to keep the distinction between hot and cold Signal at the type level, so RAC offers both the `Signal` and `SignalProducer` types.

The `Signal` type is defined as follows:

```swift
class Signal<T, E: ErrorType> {
  ...
}
```

Signal is parametrized over `T`, the type of the events emitted by the signal and `E`, the type that denotes the errors.

A Signal can be created by passing to the initializer a generator closure, which is then invoked with a sink of type `SinkOf<Event<String, NoError>>`. The sink is then used by the closure to forward events to the Signal through the `sendNext( )` method. A similiar approach has been implemented also by Akka Streams.

**In many cases, a stream of events that has no start and no end can't terminate with an error.** A simple example of this case is a stream of press events from a button. To overcome this possibility, RAC introduces the type `NoError`, meaning that the signal can't error out.

For the "button example", the type for the related signal might be `Signal<Void, NoError>`, that means:

- the user of the signal only cares about the occurence of the event, no further information will be provided
- the signal can't error out, since in this particular case it has no sense to model the fact that a button press can fail

The button example is just one of many others. If we consider the amount of events that come from the UI, a lot of trivial examples come out quickly:

- a signal that models the changes of a text field
- a signal that models the arrival of notifications (remote, local, etc...)
- any other signal created combining other signals
- ...

An example for the creation of a Signal is the following, taken from a blog post of Colin Eberhardt, where a new String is produced every second.

```swift
func createSignal() -> Signal<String, NoError> {
  var count = 0
  return Signal {
    sink in
    NSTimer.schedule(repeatInterval: 1.0) { timer in
      sendNext(sink, "tick #\(count++)")
    }
    return nil
  }
}
```

To attach some side effect at each `Next`, `Error` or `Completed` event that is produced by a `Signal`, an observer has to register its interest using **observe**. The `observe` operator accepts some closures or functions for any of the event types the user of the API is interested in.

The real deal in using Signals is their power in term of declarativeness when combining Signals with operators to create new Signals to work with.

In RAC, all **operations** that can be applied to signals are simply **free functions**, in contrast to the "classical-method-definition-on-the-Signal-type approach". As an example, the `map` operator signature is defined as follows, with the Signal on which the transformation will be applied passed as an argument:

```swift
public func map<T, U, E>(transform: T -> U)
    (signal: Signal<T, E>) -> Signal<U, E> {
    ...
}
```

To keep the APIs fluent RAC also introduces the pipe-forward operator `|>`, defined as follows:

```swift
public func |> <T, E, X>(signal: Signal<T, E>,
    transform: Signal<T, E> -> X) -> X {
	return transform(signal)
}
```

The `|>` operator doesn't do anything special, it only creates a specification.

RAC already offers a numbers of built in operators as free functions, such as `combineLatest`, `zip`, `takeUntil`, `concat`, ...

A complete but simple example of usage of all of the aspects introduced is the following:

```swift
createSignal()
  |> map { $0.uppercaseString }
  |> observe(next: { println($0) })
```

The beauty of this approach is that the chain of operations fits the types of each operation, so when the code compiles (and if the user of the APIs has learnead the semantics of each operators) the computation acts as expected.
All the relevant work for the newcomers of the paradigm consist in taking the time needed to learn the basics and play with the types and the operators.

## SignalProducer

The previous section introduced `Signal` as the type that implements the "hot signal" abstraction. This section will cover the other half of the story.

In RAC, **"cold signals"** are implemented with the `SignalProducer` type, defined as follows:

```swift
public struct SignalProducer<T, E: ErrorType> {
    ...
}
```

The generic types are the same as its "hot" counterpart, and also the initializer is pretty similar, with a generator closure:

```swift
func createSignalProducer() -> SignalProducer<String, NoError> {
  var count = 0
  return SignalProducer {
    sink, disposable in
    NSTimer.schedule(repeatInterval: 0.1) { timer in
      sendNext(sink, "tick #\(count++)")
    }
  }
}
```

Using a popular philosophical metaphore, again:

>If a tree falls in a forest and no one is around to hear it, **it doesn't make a sound**.

Or, in other words, if no one subscribes to the SignalProducer, nothing happens. For SignalProducer, the terminology for subscribing to its events is `start( )`.

If more than one observer subscribe to the same SignalProducer, the resources are allocated for each observer. In the example above, every time an observer invoke the `start( )` method on the same SignalProducer instance, a new istance of NSTimer is allocated.

Also on SignalProducers can be applied a wide range of operators. RAC doesn't implement all the operators twice for Signal and SignalOperator, but it offers a pipe-forward operator that lifts the operators and transformation that can be applied to `Signal` to also operate on `SignalProducer`.

The implementation of `|>`, that applies on the SignalProducer type using Signal's operator, is the following:

```swift
public func |> <T, E, U, F>(producer: SignalProducer<T, E>,
      transform: Signal<T, E> -> Signal<U, F>) -> SignalProducer<U, F> {
  return producer.lift(transform)
}
```
