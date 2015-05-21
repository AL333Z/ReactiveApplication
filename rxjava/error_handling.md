# Error handling

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
