# Reactive confusion

In these days **reactive programming** and **functional reactive programming** are two terms that create a lot of **confusion**, and maybe are also over-hyped.

The definitions that come from Wikipedia are still pretty vague and only Functional reactive programming introduced by Connal Elliott has a clear and simple denotative semantics. In advance to this, Elliott himself doesn't like the name he gave to the paradigm.

The concepts behind these newly introduced paradigms are prerry similar. Reactive programming is a paradigms that facilitates the development of applications by providing **abstractions** to express *time-varying values* and automatically propagating the *changes*: behaviors and events.
Functional reactive programming can be considered a "sibiling" of reactive programming, providing composable abstractions. Elliot identifies the key advantages of the functional reactive programming paradigm as: *clarity, ease of construction, composability, and clean semantics*.

In 2013 Typesafe launched http://www.reactivemanifesto.org, which tried to define what reactive applications are. The reactive manifesto didn't introduce a new programming paradigm nor depicted RP as we intended RP in this thesis. What the reactive manifesto did instead was to depict some really relevant computer science principles about scalability, resilience and event-driven architectures, and RP is one of the tools of the trade.

Erik Meijer, in his talk "Duality and the End of Reactive", concluded that all the hype and the buzzwords around the "reactive world" has no sense, and that the core of the paradigm is **all about composing side effects**. In his talk and related papers, he depicted the **duality** that links enumerables and observables.

| | One | Many |
| -- | -- | -- |
| Synchronous | T/Try[T] | Iterable[T] |
| Asynchronous | Future[T] | Observable[T] |

In short, an enumerator is basically a getter with the ability to fail and/or terminate. It might also return a future rather than a value. An **enumerable** is a getter that returns an **enumerator**. If we take the category-theoretic dual of these types we get the **observer** and **observable** types. And this conclusion is what let us to relate all the principal effects in programming, where in one axis there's is the nature of the computation (sync or async) and in the other one there's the cardinality of the result (one or many).
