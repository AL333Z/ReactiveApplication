# Publisher

The **Publisher** interface has only one method to impelement, and is as follows.

```
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

A subscriber is subscribed to a publisher via the `subscribe( )` method. This method doesn't return a `Subscription` as one would expect, but `Unit`. This is a precise design choice, since every method of the interfaces returns `Unit`.

The publisher, after notifying the subscriber that it has been subscribed via its `onSubscribed( )` method, must provide items to the subscriber, that will receive items in its `onNext( )` method.
The items received must not exceed the total number of items that the subscriber has signalled demand for.

When the subscriber `cancel( )` the subscription, the publisher must start sending items.

Finally, a pubblisher must notify to the subsciber through `onError( )` and `onComplete( )` methods if an error is encountered or the stream is successfully completed respectively.
