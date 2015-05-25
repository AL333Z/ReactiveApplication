# Scheduler

In the previous sections a lot of concepts have been introduced. This section will cover one of the most important aspects of the framework: **schedulers**.

A scheduler is an *object that schedules unit of work*, and it's implemented throgh the the `Scheduler` type. This type allows to specify **in which execution context** the chain or part of the chain has to run. In particular, the developer can choose in which thread:

- an `Observable` has to run, with `subscribeOn( )`
- a `Subscriber` has to run, with `observeOn( )`

The framework already provide some schedulers:
  - `immediate( )`, that executes work **immediately** on the current thread
  - `newThread( )`, that creates a **new Thread** for each job
  - `computation( )`, that can be used for **event-loops**, processing callbacks and other computational work
  - `io( )`, that is intended for **IO-bound** work, based on an Executor thread-pool that will grow as needed
  - ...

A really nice feature is the fact that applying the execution of a chain to a particular scheduler doesn't break the chain of operators, keeping the code clean and with a good level of declarativness.

An example of usage of schedulers is the following, in which an image is fetched from the network and then processed. A network request is a typical io-bound operation and it's performed in `io()` scheduler, while a processing operation is a cpu-bound operation and it's performed in a `computation()` scheduler.

```java
myObservableServices.retrieveImage(url)
  .subscribeOn(Schedulers.io())
  .observeOn(Schedulers.computation())
  .subscribe(bitmap -> processImage(bitmap));
```
