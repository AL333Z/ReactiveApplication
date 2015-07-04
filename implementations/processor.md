# Processor

A **Processor** represents a processing stage, which is both a Subscriber and a Publisher and obeys the contracts of both. The interface is defined as follows.

```
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> { }
```

A processor is an intermediate abstraction that enables to build **chain of processing stage**, in which each intermediate unit is a processor that can consume, transform and publish items.
The importance of this abstraction is in the fact that, obeying to both Subscriber and Publischer interface, it has to **retain back-pressure propagation** with the original stream source.
