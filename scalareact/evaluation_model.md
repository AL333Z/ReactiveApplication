# Evaluation model

Scala.React’s propagation model is **push-driven**, and use a **topologically ordered dependency graph**. This implementation detail ensures that an expression is always evaluated after all its dependants have been evaluated, so glitches can't happen.

Scala.React proceeds in **propagation cycles**. The system is *either in a propagation cycle or*, if there are no pending changes to any reactive, *idle*. The model of **time** the system use is a **discrete** one.

Every propagation cycle has two phases: first, all reactives are synchronized so that observers, which are run in the second phase, cannot observe inconsistent data. During a propagation cycle, the reactive world is paused, i.e., no new changes are applied and no source reactive emits new events or changes values.

A propagation cycle proceeds as follows:

1. Enter all modified/emitting reactives into a priority queue with the priorities being the reactives’ levels.
2. While the queue is not empty, take the reactive on the lowest level and validate its value and message. The reactive decides whether it propagates a message to its dependents. If it does so, its dependents are added to the priority queue as well.
