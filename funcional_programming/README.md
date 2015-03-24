# Funcional programming

From wikipedia:
> In computer science, functional programming is a programming paradigm, a style of building the structure and elements of computer programs, that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data. It is a declarative programming paradigm, which means programming is done with expressions. In functional code, the output value of a function depends only on the arguments that are input to the function, so calling a function f twice with the same value for an argument x will produce the same result f(x) each time. Eliminating side effects, i.e. changes in state that do not depend on the function inputs, can make it much easier to understand and predict the behavior of a program, which is one of the key motivations for the development of functional programming.

Functional programming has its roots in Lambda Calculus, defined by Alonzo Church in the 1930s, that provides a conceptual framework for describing functions and their evaluation.

Functional programming concepts have been around for many year, but in the last few years the topic has gained a brand new popularity. The main reason is the progress and the spread of multi-core processors, that led to a greater chance for developers to use and put in practice concurrent and parallelized implementations. Infact, traditional imperative programming constructs (based on side effects) are not a good fit, since it's really hard to reason about in such environment.
Functional languages are more supportive in these environments, since they offer constructs that simplify time reasoning about what applications are doing at any given point in time.

What follows next is a short overview of concepts that can be derived from Wikipedia's definition and some further explanations about other crucial topics.

## Model of computation

The **model of computation** is represented by **mathematical functions** and **expression evaluation**. This is in contrast with the classic semantics of functions as sets of arguments/value pairs.

## First-class and high order functions
Functions can be seen as **first-class values** of the language. In particular, they can be **passed as arguments** to other functions, **returned as a results** and even **stored** in data structures.
A brilliant example of this concept is the differential operator $${d \over dx}$$ from mathematics.

Formally, the term *high-order* describes the concept of a *function that operates on other functions*. The term *first-class* describes programming language entities that *have no restriction on their use*, instead.

High order functions allows to increase modularity in programs, by serving as a mechanism for glueing program fragments togheter, through composition and also abstraction over functional behaviors.

## No explicit state, no side effects
From Wikipedia's definition of side effect:
>In computer science, a function or expression is said to have a side effect if, in addition to returning a value, it also modifies some state or has an observable interaction with calling functions or the outside world.

Side effects are a possible mean that can help to keep track of system's state. Typical examples of side effects are memory and I/O operations.
*In purely functional functions there're no side effects at all*. This means that state, if needed, must be carried explicitly as a function argument.

The most common way to denote *actions* in *functional terms* is through the **Monad** abstraction. The monad is an abstraction that doesn't compromise the purity of the functional approach.
A more detailed explanation about these arguments will be provided in the next sections of this chapter.

## Declarativeness
Functional programming has an higher level of declarativeness than imperative programming, better expressing *what* and abstracting from *how*.
