# SignalProducer

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
