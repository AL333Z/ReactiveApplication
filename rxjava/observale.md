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

## Operator

In RxJava, operators are what enable the devolper to **model** the actual computation. An operator allows to perform **almost every type of manipulation** on the source observer in a declarative way.

Expressing a computation in terms of a stream of values is translated in building a **chain of proper operators**. Usually, looking at the signatures and at the types of the operators  is really helpful when choosing which operator is the right one for the goal to achieve.

An operator, to be applicable to an Observable, has to implement the `Operator` interface and has to be lifted. The `lift` function lifts a function (inside an `Operator`) to the current Observable and returns a new Observable that when subscribed to will pass the values of the current Observable through the `Operator` function.

Operators are methods of the `Observable` class, so creating a chain of operators starting from a source observable is a pretty straightforward process.

RxJava provides a huge sets of operators, and a lot of them is defined in terms of other ones. What follows is only a small introductive subset.

## Map

`Map` is an operator that returns an Observable that **applies a specified function to each item** emitted by the source Observable and emits the results of these function applications. Its marble diagram is the following.

![Map operator](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/map.png)

To better clarify the concept of lifting introduced previously, let's also look at the definition and implementation for `Map`.

```java
public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
  return lift(new OperatorMap<T, R>(func));
}

...

public final class OperatorMap<T, R> implements Operator<R, T> {

    private final Func1<? super T, ? extends R> transformer;
    public OperatorMap(Func1<? super T, ? extends R> transformer) {
        this.transformer = transformer;
    }

    public Subscriber<? super T> call(final Subscriber<? super R> o) {
        return new Subscriber<T>(o) {

            public void onCompleted() {
                o.onCompleted();
            }

            public void onError(Throwable e) {
                o.onError(e);
            }

            public void onNext(T t) {
                try {
                    o.onNext(transformer.call(t));
                } catch (Throwable e) {
                    onError(OnErrorThrowable.addValueAsLastCause(e, t));
                }
            }
        };
    }
}
```

## FlatMap

`FlatMap` returns an Observable that **emits items based on applying a function that is supplied to each item emitted by the source Observable, where that function returns an Observable**, and then merging those resulting Observables and emitting the results of this merger.

![FlatMap operator](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/flatMap.png)

## Filter

`Filter` is quite obvious.

![Filter operator](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/filter.png)

## Scan

`Scan` returns an Observable that **applies a specified accumulator function** to the first item emitted by a source Observable, then **feeds the result of that function along** with the second item emitted by the source Observable into the same function, and so on until all items have been emitted by the source Observable, and emits the final result from the final call to your function as its sole item.

![Scan operator](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/scan.png)

## Take

`Take` returns an Observable that emits only the first n items emitted by the source Observable.

![Take operator](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/take.png)

## A complex example

```java
// Returns a List of website URLs based on a text search
Observable<List<String>> query(String text) { ... }

// Returns the title of a website, or null if 404
Observable<String> getTitle(String URL){ ... }

query("Hello, world!")                    // -> Observable<List<String>>
  .flatMap(urls -> Observable.from(urls)) // -> Observable<String>
  .flatMap(url -> getTitle(url))          // -> Observable<String>
  .filter(title -> title != null)
  .doOnNext(title -> saveTitle(title))    // extra behavior
  .map(title -> new Pair<Integer, String>(0, title)) // -> Observable<Pair<Integer, String>>
  .scan((sum, item) -> new Pair<Integer, Word>(sum.first + 1, item.second))
  .take(5)
  .subscribe(indexItemPair ->
      System.out.println("Pos: " + indexItemPair.first + ": title:" + indexItemPair.second ));
```

The example starts with the hyphotesis of having two methods that returns observable, for example coming from the network layer of an application. `query` return a list of url given a text and `getTitle` returns the title of a website or null.

The computation aims to return all the title of the websites that match the "Hello, World!" string.

The code itself is pretty self-explanatory, and shows how concise and elegant a computation can be using the approach suggested by RxJava in respect to its imperative-style counterpart.

## Error handling

The previous sections introduced the basics of `Observable` and `Operator`. This section will introduce how errors are handled in RxJava.

As introduced previously, every Observable ends with either a single call to `onCompleted()` **or** `onError()`.

What follows is an example of a chain of operators that contains some transformation that may also fail.

```java
Observable.just("Hello, world!")
  .map(s -> potentialException(s))
  .map(s -> anotherPotentialException(s))
  .subscribe(new Subscriber<String>() {
      @Override
      public void onNext(String s) { System.out.println(s); }

      @Override
      public void onCompleted() { System.out.println("Completed!"); }

      @Override
      public void onError(Throwable e) { System.out.println("Ouch!"); }
    });
```

The `onError()` callback is called if an `Exception` is thrown **at any time** in the chain, thus the operators don't have to handle exceptions in first place since they are **propagated** to the `Subscriber`, which has to manage all the error handling.
