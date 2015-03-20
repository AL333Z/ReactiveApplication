# Failure handling

In distribuited systems a big issue isn't *if* a failure occurs, but only *when* or *how often*. Failures are certain.
Reactive Applications principles differentiate **resilience** and **reliability**, if favour of the first one.

Wikipedia defines Resilience as:
> the ability [of a system] to cope with change.

In the context of system enginering change is also represented by failures. Resilient system goes one step further than fault tolerance systems, *recovering* theirs original *shape* and *feature* set.

One of the key points in Reactive Application is the principle of *isolation* and *loose coupling* of components. Having isolated and compartmentalized components helps avoiding cascading failures that compromise the whole system.

In this scenario, a consequence immediately arise: the failed compartment itself should not be responsible for its own recovery. This new requirement may be achieved by introducing failure handling delegation, through **supervision**.
Supervision means that normal requests and responses flow separately from failures: while the former are exchanged between the user and the service, the latter travel from the service to its supervisor. This distinction enables decoupling the core-logic of the system (or the "happy path") from the management and the recovery of the failures.

![A representation of supervision. Image from the book "Reactive Design Patterns"](https://dl.dropboxusercontent.com/u/5045423/supervision.png)

Usually, given a component and its failures, the decision about how to manage and recover are *escalated* to its supervisor.
The chapter about *the implementations that supports Reactive* will go deeper, and explore this concepts, pragmatically.

From another point of view, having isolated components implicitly requires the interaction between each components to be **asynchronous**. The next section will examine how this can be achieved thanks to event orientation.
