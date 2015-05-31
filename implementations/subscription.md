# Subscription

A **Subscription** abstracts the notion of a subscriber's communication channel back to the publisher. This channel is what enables the subscriber to either cancel the subscription or signal demand.
Its definition is as follows.

```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```

The `request( )` method signals to the publisher that the subscriber can receive more items, also specifying the quantity.

The `cancel( )` method notifies the publisher that the subscriber is no longer interested in receiving items.

This channel between a publisher and a subscriber is what enables to achieve non blocking back-pressure in a Reactive Streams.
