# Action

The last concept that RAC APIs introduce is the notion of `Action`.

An action is something that will do some work in the future. An action will be executed with an input and will return an output or an error. Its type is generic, and it's exposed as:

```swift
public final class Action<Input, Output, Error: ErrorType>
```

The constructor of an Action accepts a closure or a function that creates a SignalProducer for each input, with the type `Input -> SignalProducer<Output, Error>)`.

A pratical and useful use of Action is in conjunction with `CocoaAction`, that is another type that wraps an Action for use by a GUI control, with key-value observing, or with other Cocoa bindings.

