# Functional domain modelling

In functional domain modelling, a system may be represented through the following main domain elements: entities, value objects and services.

## Entities and value objects
Each *entitiy* has a unique identity. An entity may have many attributes, and each attribute may changes in course of the lifetime of the system. When an attribute within an entity changes, the identity itself is still the same. In this meta-model, attributes are abstracted by *value objects*.

A value object is immutable by definition and can be freely shared across entities. This doesn't mean that an entities's attributes can't change.
To clear this concept, let's consider the following example, with an entity person within a system, which is univocally identified by an unique id and that has an "address" attribute. Now the difference between the two concepts is immediate: when we talk of a person we usually refer to a *specific instance* of person, while when we talk of an address we just refer to the *value part*.

Functional programming aims to model **as much immutability as possible**, so a possible approach to model entities and attributes is the following:
- remember that an **entity** has an identity that cannot change, so it's **semantically mutable**
- remember that a **value object** has a value that cannot change, so it's **semantically immutable**

The fact that an entity is semantically mutable doesn't prevent us from using immutable constructs for its implementation.

## Services

The last element of this meta-model is the **service**, that is a more macro level abstraction than entity and value object. In short, a service models a use case, encapsulating a complete operation that has a certaing value to user of the system.

A service usually involves multiple entities and value objects, that interact according to specific business rules to deliver specific functionalities.
