# ProperyType

The previous sections introduced the abstractions that RAC offers to describe signal. This section will introduce other collateral but usefull types.

`PropertyType` is a protocol that, when applied to a property, **allows the observation of its changes**. Its definition is as follows:

```swift
public protocol PropertyType {
	typealias Value

	var value: Value { get }
	var producer: SignalProducer<Value, NoError> { get }
}
```

The semantics of this protocol is neat:
- the **getter** return the current value of the property
- the **producer** return a producer for Signals that will send the property's current value followed by all changes over time

Starting from this protocol, RAC introduces:
- `ConstantProperty`, that represents a property that never change
- `MutableProperty`,  that represents a mutable property
- `PropertyOf`, that represents a read-only view to a property

These types are really usefull when used in combination with the `<~` operator, that binds properties together. The bind operator comes in three flavors:

```swift
/// Binds a signal to a property, updating the property's value to the latest
/// value sent by the signal.
public func <~ <P: MutablePropertyType>(property: P, signal: Signal<P.Value, NoError>) -> Disposable {}

/// Creates a signal from the given producer, which will be immediately bound to
/// the given property, updating the property's value to the latest value sent
/// by the signal.
public func <~ <P: MutablePropertyType>(property: P, producer: SignalProducer<P.Value, NoError>) -> Disposable { }

/// Binds `destinationProperty` to the latest values of `sourceProperty`.
public func <~ <Destination: MutablePropertyType, Source: PropertyType where Source.Value == Destination.Value>(destinationProperty: Destination, sourceProperty: Source) -> Disposable { }
```

What these operators do is to create the wires that link each property to each others, in a declarative manner.
Each property is observable, through its inner SignalProducer.

