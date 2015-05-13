# Signal

As introduced previously, Signal implements the abstraction of **hot signal** as a signal that typically has **no start and no end**.

A typical pratical example for this type can be represented by the press events of a button or the arrive of some notifications. They are just events that happens with no relevance of the fact that someone is observing them.

Using a popular philosophical metaphore:

>If a tree falls in a forest and no one is around to hear it, **it does make a sound**.

Looking at the documentation, a Signal is defined as a *push-driven stream that sends Events over time*. Events will be sent to all observers at the same time. So, if a Signal has different observers subscribed to its events, every observer will always see the same sequence of events.

The documentation says another crucial thing, that clarifies the choice of abstration made by the developers:

> Signals are generally used to represent event streams that are already "in progress", like notifications, user input, etc. To represent streams that must first be _started_, see the SignalProducer type.

The developers of RAC decided to keep the distinction between hot and cold Signal at the type level, so RAC offers both `Signal` and `SignalProducer` types.

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

To use Signals, for example to attach some side effect at each `Next`, `Error` or `Completed` event that is produced, the terminology for the `Signal` type is **observe**. RAC provides the `observe` method, that accepts closures or functions for any of the event types the user of the API is interested in.

The real deal in using Signals is their power in term of declarativeness when combining Signals with operators to create new Signals to work with.

In RAC, all **operations** that can be applied to signals are simply **free functions**, in contrast to the "classical-method-definition-on-the-Signal-type approach". As an example, the `map` operator signature is defined as follows, with the Signal on which will be applied the transformation passed as an argument:

```swift
public func map<T, U, E>(transform: T -> U)
    (signal: Signal<T, E>) -> Signal<U, E> {
    ...
}
```

RAC mantainers, to keep the APIs fluent, also introduced the pipe-forward operator `|>`, defined as follows:

```swift
public func |> <T, E, X>(signal: Signal<T, E>,
    transform: Signal<T, E> -> X) -> X {
	return transform(signal)
}
```

The `|>` operator doesn't do anything, it only creates a specification.

RAC already offers a numbers of built in operators as free functions, such as `combineLatest`, `zip`, `takeUntil`, `concat`, ...

A complete but simple example of usage of all of the aspects introduced is the following:

```swift
createSignal()
  |> map { $0.uppercaseString }
  |> observe(next: { println($0) })
```

The beauty of this approach is that the chain of operations fits the types of each operation, so when the code compile (and if the user of the APIs has learnead the semantics of each operators) the computation acts as expected.
All the relevant work for the newcomers of the paradigm is in taking time to try and play with the types and the operators.

