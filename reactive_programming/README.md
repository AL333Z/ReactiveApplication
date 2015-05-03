# Reactive programming

When using the imperative programming paradigm, the act of capturing dynamic values is done only indirectly, through **state and mutations**. In this context, the idea of *dynamic/evolving* values is not a first class value in the paradigm. Moreover, the paradigm can only capture discrete evolving values, since the paradigm itself is *temporally discrete*.

Reactive programming is a programming paradigm that aims to provide a more declarative way to abstract computations and mutations.

Wikipedia defines reactive programming as:
>a programming paradigm oriented around data flows and the propagation of change. This means that it should be possible to express static or dynamic data flows with ease in the programming languages used, and that the underlying execution model will automatically propagate changes through the data flow.

In this definition emerges some key concepts:
- expressing computations in terms of **data flows**
- **change is propagated** in a composable way
- the language or framework has to support **declarativeness**
- all the "dirty work" is done by the execution model, ensuring that the actual computation respects the semantics

This chapter will focus on **Functional Reactive Programming** and **Reactive Programming**, in an attempt to provide formal definitions.
