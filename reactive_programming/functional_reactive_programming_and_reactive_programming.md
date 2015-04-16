# Reactive confusion

In these days **Reactive Programming** and **Functional Reactive Programming** are two terms that create a lot of **confusion**, and maybe are over-hyped.
The definitions that come from Wikipedia are still pretty vague and only Functional Reactive Programming introduced by Connal Elliott has a clear and simple denotative semantics. From another point of view, Elliott himself doesn't like the name he gave to the paradigm.

The concepts behind these newly introduced paradigms are prerry similar. Reactive Programming is a paradigms that facilitates the development of applications by providing **abstractions** to express *time-varying values* and automatically propagating the *changes*: behaviors and events.
Functional Reactive Programming can be considered a "sibiling" of Reactive Programming, providing composable abstractions. Elliot identifies the key advantages of the Functional Reactive Programming paradigm as: *clarity, ease of construction, composability, and clean semantics*.

In 2013 Typesafe launched http://www.reactivemanifesto.org, which tried to define what reactive applications are. The reactive manifesto didn't introduce a new programming paradigm nor depicted Reactive Programming. It just depicted some computer science principles about scalability, resilience and event-driven architectures.

Erik Meijer, in his talk "Duality and the End of Reactive", concluded that all the hype and the buzzwords around the reactive world has no sense, and that the core of the paradigm is **all about composing side effects**. In his talk and related papers, he depicted the **duality** that links enumerables and observables. In short, an enumerator is basically a getter with the ability to fail and/or terminate. It might also return a promise rather than a value. An enumerable is a getter that returns an enumerator. If we take the category-theoretic dual of these types we get the observer and observable types. And this conclusion is what let us relate all the principal effects in programming.
