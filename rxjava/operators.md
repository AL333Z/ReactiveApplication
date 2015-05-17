# Operators

In RxJava, operators are what enable the devolper to **express** the actual computation. An operator allows to perform **almost each type of manipulation** on the source observer.

Expressing a computation in terms of a stream of values is translated in building a **chain of proper operators**. Usually, looking at the signatures of the operators and at the types is really helpful when choosing which operator is the right one for the goal to achieve.

An operator, to be applicable to an Observable, has to implement the `Operator` interface and has to be lifted. The `lift` function lifts a function (inside an `Operator`) to the current Observable and returns a new Observable that when subscribed to will pass the values of the current Observable through the `Operator` function.

Operators are methods of `Observable` class, so creating a chain of operators starting from a source observable is a pretty straightforward process.

RxJava provides a huge sets of operators, and a lot of them is defined in terms of others. What follows is only a small introductive subset.

## Map

`Map` is an operator that returns an Observable that **applies a specified function to each item** emitted by the source Observable and emits the results of these function applications. Its marble diagram is the following.

![](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/map.png)

To better clarify the concept of lifting, let's also look at its definition and implementation.

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

![](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/flatMap.png)

## Filter

`Filter` is quite obvious.

![](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/filter.png)

## Scan

`Scan` returns an Observable that **applies a specified accumulator function** to the first item emitted by a source Observable, then feeds the result of that function along with the second item emitted by the source Observable into the same function, and so on until all items have been emitted by the source Observable, and emits the final result from the final call to your function as its sole item.

![](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/scan.png)

## Take

`Take` returns an Observable that emits only the first n items emitted by the source Observable.

![](https://raw.githubusercontent.com/AL333Z/RxAndroid-overview/master/images/take.png)

## A complex example

```java
// Returns a List of website URLs based on a text search
Observable<List<String>> query(String text);

// Returns the title of a website, or null if 404
Observable<String> getTitle(String URL);

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

The code itself is pretty self-explanatory, and shows how concise and elegant a computation can be using the approach suggested by RxJava in respect to its imperative-style counterpart.
