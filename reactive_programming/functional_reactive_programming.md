# Overview

Functional reactive programming has its origin with *Fran* (Functional reactive animation), a Haskell library by Conal Elliott for interactive animations.
Fran's author found it difficult to express the *what* of an interactive animation abstracting from the *how* and built a set of expressive and recursive data types, combined with a declarative programming language.

Informally, **functional reactive programming** is a programming paradigm which brings a notion of **time** in the functional programming paradigm.

Conal Elliott says that FRP is all about two main things: *denotative* and *temporally continuos*. Infact, he also likes the term "*denotative, continuos-time programming*" to replace functional reactive programming, since it reduces the confusion.

Always about the **denotative** part, he means that the paradigm should be founded on a precise, simple, implementation-independent, compositional semantics that exactly specifies the meaning of each type and building block. The compositional nature of the semantics then determines the meaning of all type-correct combinations of the building blocks.

>For me, denotative is the heart and essence of functional programming, and is what enables precise and tractable reasoning and thus a foundation for correctness, derivation, and optimization

About the **continuous time** part, there's some confusion. Some claim that it's an idea somehow unnatural or impossible to implement considering the discrete nature of computers. To this issues, Elliott answers in the following way:

>This line of thinking strikes me as bizarre, especially when coming from Haskellers, for a few reasons:
- Using lazy functional languages, we casually program with infinite data on finite machines. We get lovely modularity as a result [...].
- There are many examples of programming in continuous space, for instance, vector graphics, [...]
- I like my programs to reflect how I think about the problem space rather than the machine that executes the programs, and I tend to expect other high-level language programmers to share that preference.

Another name that Elliott suggests for continuous is *resolution-independent*, and thus able to be transformed in time and space with ease and without propagating and amplifying sampling artifacts. As an example, he propose the "finite vs infinite" data structure issue:

>We only access a finite amount of data *in the end*. However, allowing infinite data structures *in the middle* makes for a much more composable programming style. Each event has a stream (finite or infinite) of occurrences. Each occurrence has an associated time and value.

#Behaviors and Event Streams

Fran introduces two special abstraction
- **behaviors**, values that are continuos functions of time.
- **event streams**, values which are discrete functions of time.

**Behaviors** are dynamic/evolving values, and are first class values in themselves. You can define them and combine them, pass them into and out of functions. Behaviors are built up out of a few primitives, like constant (static) behaviors and time (like a clock), and then with sequential and parallel combination. n-behaviors are combined by applying an n-ary function (on static values), "point-wise", i.e., continuously over time.

Examples of behaviours are the following:
- time
- a browser window frame
- the cursor position
- the position of a image during an animation
- audio data
- ...

**Events** enable to account for discrete phenomena, and each of which has a stream (finite or infinite) of occurrences. Each occurrence has an associated time and value. Formally, the points at which an event stream is defined are termed *events* and events are said to *occur* on an *event stream*.

Example of event streams are the following:
- timer events
- key presses
- mouse clicks
- MIDI data
- network packets
- ...
