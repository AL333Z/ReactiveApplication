# Error handling

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

Every Observable ends with either a single call to `onCompleted()` **or** `onError()`.

- `onError()` is called if an `Exception` is thrown **at any time**
- the operators don't have to handle exceptions
- exceptions are **propagated** to the `Subscriber`, which has to manage all the error handling

---
