# A conclusive comparison

This thesis started with an overview of RP, that introduced the foundations and the main aspects of the paradigm, and then continued with a detailed overview of some of the most popular or relevant frameworks that allow the application of the paradigm in different platforms and languages, with also some pratical use cases.

This final section will attempt to wrap everything up, with a conclusive comparison of each library peculiarities and the approach to RP that each library suggests.

## Operators, Expressions, Declarativness

In regards to **expressing a computation in terms of a flow of intermediate steps**, the approach that Scala.React suggests with its **signal expressions** is the main difference in respect to all other approaches, that are instead based on the presence of **operators**.
Scala.React's Signal type has no operator at all, and lets the developer to build a "reactive" computation with an imperative-style code, at least in appearance. Infact, signal expressions and the magic behind Var and Val implicit conversion looks natural to a delevoper with an object oriented background.

All the other libraries introduce an approach based on operators, indeed:
- RxJava offers a wide range of **methods** in the `Observable` type
- ReactiveCocoa offers a wide range of **free functions**, used in combination with the pipe-forward operator `|>` on the `Signal` and `SignalProducer` type to give the user an elegant way to build a chain of operators without loosing the purity of the approach based on free functions
- Akka Strams offers a (still) minimal but effective set of **components**, with the foundamental abstraction defined in terms of the `Flow`, `Sink` and `Source` type, and the ability to build linear or graph computations simply using the to `~>` operator.

All the library previously introduced presented an high level of declarativness, with a good level of intregation in the ecosystem they belong to:
- Scala.React does an amazing job in solving the issues of the "standard" observer pattern leveraging on the construct offered by the Scala language.
- RxJava offer a pretty straigh-forward implementation of Rx to the Java ecosystem. Both the implementation and the interfaces really benefitted by the advent of Java 8 and lambda expression (or, in Android, Retrolambda) in the language ecosystem.
- RAC has come a long way from its Objective-C years, and now, with Swift and the current 3.0 version, offers a nice and clean set of interfaces and abstractions.
- Akka Streams is, in this thesis author's hopinion, the most impressive approach to modelling computation declaratively. The approach of building a graph-resembling DSL to express not-linear computation is a feature that really helps in keeping the code clean and intuitive, even if the initial learning curve and some of the magic behind the library is a thing to always keep in consideration when evaluating a tool.

## Hot and Cold, Push and Pull, Back-pressure

Another aspect that distinguish the libraries is the way they abstract the notion of who actually starts the computation, and thus, the propagation of changes.

In this regards, RAC offer a pretty straight forward abstraction putting this distinction directly at type level: the `Signal` type is producer-driven, so the changes are pushed to the subscribers as they happens/are produced, while the `SignalProducer` type is demand driven, so the changes are pulled.

In RxJava this distinction is not obvious, but there're a set of types and operators that helps when building observables that should be producer-driven or demand-driven. For example, the `Subject` type is usually used to "warm" a cold observable into an hot observable. RxJava is also conform to Reactive Streams, just like Akka Streams.
Being conform to Reactive Streams means that for both libraries is valid the principle that a producer can never send more items than a consumer can handle, using the so called "dynamic push-pull" approach.

In practice, the usage of this feature really depends from the use case to satisfy. For example, for Akka Streams and RxJava the conformation to Reactive Streams was a pretty straight forward process (engineers from Netflix, Typesafe, etc.. all find it problematic having a consumer overwhelmed by a production of items they can handle), since those frameworks are used to process an high number of items and also to build infrastructures and services.
In the context of mobile applications, currently RAC doesn't directly solve this issue, since RAC mantainers believe that figuring out the behavior of side effects is far more important, since effects dominate GUI applications.


```````
