# Subject

**Subject** is a further entity that is provided by the framework.
A `Subject` is a sort of bridge or proxy that acts both as an observer and as an `Observable`:
- because **it is an observer**, it can **subscribe** to one or more observables
- because **it is an Observable**, it can pass through the items it observes by **reemitting** them, and it can also **emit** new **items**

Subject has the "power" of **turning a cold observable hot**. Infact, when a `Subject` subscribes to an `Observable`, it will trigger that `Observable` to begin emitting items (and if that `Observable` is "cold" — that is, if it waits for a subscription before it begins to emit items). This can have the effect of making the resulting `Subject` a "hot" `Observable` variant of the original "cold" `Observable`.

The framework provides a wide range of subject, each with its semantics. What follows is an overview of the main popular and used.

NB: the images have been taken from the documentation.

## PublishSubject

![PublishSubject](http://reactivex.io/documentation/operators/images/S.PublishSubject.e.png)

`PublishSubject` emits to an observer only those items that are emitted by the source Observable(s) **subsequent** to the time of the subscription. This means that an observer will not receive the previous emitted items.

## ReplaySubject

![ReplaySubject](http://reactivex.io/documentation/operators/images/S.ReplaySubject.png)

`ReplaySubject` emits to any observer **all** of the items that were emitted by the source Observable(s), regardless of when the observer subscribes. To keep the memory consuption limited, this subject also use a bounded buffer that enable to discard old items when the limit size has been reached.

## AsyncSubject

![AsyncSubject](http://reactivex.io/documentation/operators/images/S.AsyncSubject.png)

An `AsyncSubject` caches and only remember the **last** value of the Observable, and only after that source Observable completes emits that value.

## BehaviorSubject

![BehaviorSubject](http://reactivex.io/documentation/operators/images/S.BehaviorSubject.png)

When an observer subscribes to a `BehaviorSubject`, it begins by emitting the item most recently emitted by the source Observable (or a seed/default value if none has yet been emitted) and then continues to emit any other items emitted later by the source Observable(s).

In practice, it's really similar to the **Behavior** notion from Elliot's FRP.
