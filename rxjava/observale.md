# Observable

The foundamental entity of RxJava is the **Observable** type.
An observable is a *sequence of ongoing event ordered in time*.

An `Observable` can emit 3 type of items:
- values
- error
- completition event

RxJava provides a really nice documentation, with also some graphical diagrams, called marble diagrams.

![A marble diagram](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/marble-intro.png)

A marble diagram depicts how a an `Observable` and an `Operator` behave:
- observables are timelines with sequence of symbols
- operators are rectangles, that accepts in input the upper observable and return in output the lower one

In RxJava, an `Observable` is defined over a generic type `T`, the type of its values, as follows:

```java
class Observable<T> { ... }
```

An `Observable` can emit - in order - **zero, one or more values, an error or a completition event**.

An `Observable` is an **immutable** entity, so the only way to change the sequence of values it emits is through the application of an **operator** and the subsequent creation of a new `Observable`.

An entity can subscribe its interest in the values coming from an `Observalbe` through its `subsribe( )` method, that accepts one, two or three `Action` parameters (that correspond to the `onNext( )`, `onError( )` and `onComplete( )` callbacks).

The classic "Hello, World" example in RxJava and Java 8 is the following:

```java
Observable.just("Hello, world!").subscribe(s -> System.out.println(s));
```

The `Observable` type provides some convenience methods that returns an observable with the given specification. An incomplete list of these is the following:

- `just( )`: convert an object or several objects into an Observable that emits that object or those objects
- `from( )`: convert an Iterable, a Future, or an Array into an Observable
- `empty( )`: create an Observable that emits nothing and then completes
- `error( )`: create an Observable that emits nothing and then signals an error
- `never( )`: create an Observable that emits nothing at all
- `create( )`: create an Observable from scratch by means of a function
- `defer( )`: do not create the Observable until a Subscriber subscribes; create a fresh Observable on each subscription
- ...

Usually, observables created with these methods are used in conjuction with other observables and operators, to create more complex logics.

