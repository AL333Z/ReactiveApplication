# Subscriber

The **Subscriber** interface abstracts the notion of an entity that consumes items.
Its definition is as follows.

```
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```

A `Subscriber` has the canonical `onNext( )`, `onError( )`, and `onComplete()` methods. When new items are produced by the publisher, a the onNext method is invoked with the new element. If an error was raised while producing values, the publisher would then invoke the onError method with the exception. Finally, when the publisher completes its job, the onComplete method is then invoked.

The `onSubscribe( )` method is invoked when a subscriber is subscribed to a publisher. This method is really important for the framework, since it links the relation between a subscriber and a publisher, via a `Subscription`.
