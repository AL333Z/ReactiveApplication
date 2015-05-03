# Functional Reactive Programming

Functional reactive programming has its origin with *Fran* (Functional reactive animation), a Haskell library by Conal Elliott for interactive animations.
Elliott found it difficult to express the *what* of an interactive animation abstracting from the *how*, and built a set of expressive and recursive data types, combined with a declarative programming language.

Informally, **functional reactive programming** is a programming paradigm which brings a notion of **time** in the functional programming paradigm, providing a conceptual framework for implementing reactive systems. Infact, FRP let the application achieve reactivity by providing constructs for specifying *how behaviors change in response to events*.

Elliott says that FRP is all about two main things: *denotative* and *temporally continuos*. Infact, he also likes the term "*denotative, continuos-time programming*" to replace functional reactive programming, since it reduces the confusion.

Always about the **denotative** part, he means that the paradigm should be founded on a precise, simple, implementation-independent, compositional semantics that exactly specifies the meaning of each type and building block. The compositional nature of the semantics then determines the meaning of all type-correct combinations of the building blocks.

From an Elliott's quote:
>Denotative is the heart and essence of functional programming, and is what enables precise and tractable reasoning and thus a foundation for correctness, derivation, and optimization.

About the **continuous time** part, there's some confusion. Some claim that it's an idea somehow unnatural or impossible to implement considering the discrete nature of computers. To this issues, Elliott answers in the following way:

>This line of thinking strikes me as bizarre, especially when coming from Haskellers, for a few reasons:
- Using lazy functional languages, we casually program with infinite data on finite machines. We get lovely modularity as a result [...].
- There are many examples of programming in continuous space, for instance, vector graphics, [...]
- I like my programs to reflect how I think about the problem space rather than the machine that executes the programs, and I tend to expect other high-level language programmers to share that preference.

Another name that Elliott suggests for continuous is *resolution-independent*, and thus able to be transformed in time and space with ease and without propagating and amplifying sampling artifacts. As an example, he propose the "finite vs infinite" data structure issue:

>We only access a finite amount of data *in the end*. However, allowing infinite data structures *in the middle* makes for a much more composable programming style. Each event has a stream (finite or infinite) of occurrences. Each occurrence has an associated time and value.
