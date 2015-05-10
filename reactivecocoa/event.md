# Events and Signals

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
- `.Next` represents an event that carry information of type `T`
- `.Error` represents an event that carry an error of type `E`
- `.Completed` represents a successful terminal event
- `.Interrupted` represents an event that indicates that the signal has been interrupted

Just like `RxJava`'s Observable, also `RAC`'s signal can be distincted in two main categories: hot and cold signals. The main difference from RxJava's implementation is that in RAC this foundamental distinction can be found directly in the types. Infact, RAC defines `Signal`as hot and `SignalProducer` as cold signals.

*NB*: the previous introduction of Signal is general and applicable on both Signal and SignalProducer.

The next sections will go deeper and explore more of Signals and SignalProducers.
