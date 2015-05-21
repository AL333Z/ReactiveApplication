# Subscription

In RxJava, `Subscription` is an abstraction that represent the **link** between an `Observable` and a `Subscriber`.

A subscription is a quite simple type:

```java
public interface Subscription {
    public void unsubscribe();
    public boolean isUnsubscribed();
}
```

The main usage for subscription is in its `unsubscribe( )` method, that can be used to *stop* the chain, **terminating wherever it is currently executing code**.

`CompositeSubscription` is another useful type, that simplify the management of multiple and related subscriptions. A composite subscription comes with an algebra that defines the behaviors of its methods:

- `add(Subscription s)`, **adds a new Subscription** to the CompositeSubscription; if this is unsubscribed, will explicitly unsubscribing the new Subscription as well
- `remove(Subscription s)`, **removes a Subscription** from the CompositeSubscription, and unsubscribes the Subscription
- `unsubscribe()`, unsubscribes to **all** subscriptions in the CompositeSubscription
- unsubscribing inner subscriptions has no effect on the composite subscription
