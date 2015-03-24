# ADTs and Immutability

As introduced previously, functional programming doesn't allow side effects and mutable state. Mutable **state** may be easily represented by any variable that is not stable or final, and can be changed or updated inside of an application. A **mutation** is the act of updating some state in place.

A simple example that shows how fast the things become *incidentally complex* is given by the following simple example taken from a Justin Spahr-Summers's talk from Github:

```scala
var visible     // → 2 states
var enabled     // → 4 states
var selected    // → 8 states
var highlighted // → 16 states
```

In  other words, state management quickly become hard to mantain and unpredictable. By avoiding mutable state a programmer has the assurance that no one else can possibly have changed his application's state, and can reason about what a value will be at a given time.

This section will cover the basic concepts of functional domain modelling and immutable data structures, using Scala as the reference language.
