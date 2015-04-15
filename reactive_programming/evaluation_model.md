# Evaluation model

The core of languages that support Reactive Programming is about **how changes are propagated**. From the user's point of view (the programmer) everything is transparent. In other words, a change of a value is automatically propagated to all dependent computations.

At the language level this means that there should be a mechanism that is triggered when there's an event occurence at an event source. This mechanism will fire a notification about the changes to dependent computations, that will possibly trigger a recomputation, and so on.

The issue of *who initiates the propagation of changes* is a really important decision in designing a language or a framework that supports Reactive Programming.

In simple terms the question is whether the source should *push* new data to its dependents (consumers) or the dependents should *pull* data from the event source (producer).

![Push- Versus Pull-based evaluation model, from the paper "A survey on Reactive Programming"](http://i59.tinypic.com/2z3nsw5.png)

In the literature there are two main evaluation models:

- **Pull-Based**, in which the computation that requires a value has to pull it from the source. The propagation is said to be *demand-driven*. The pull model has been implemented in Fran, also thanks to the lazy evaluation of Haskell. A main trait of this approach is that the computation requiring the value has the liberty to only pull the new values when if actually needs it. From another point of view, this trait can lead to a significant latency between an occurence of an event and when its reactions are triggered.
- **Push-Based**, in which when the event source has new data it pushes the data to its dependent computations. The propagation is said to be *data-driven*. An example of a recent implementation that use this model is Scala.React (presented in the paper "Deprecating the Observer Pattern" by Maier, Rompf and Odersky).
This approach also brings the issue of wasteful recomputation, since every new data that is pushed to the consumer triggers a recomputation.

Tipically, reactive programming languages use **either a pull-based or push-based model**, but there are also languages that employ **both** approaches. In latter case there're both the benefits of the push-based model (efficiency and low latency) and those of the pull-based model (flexibility of pulling values based on demand).
