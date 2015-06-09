# Referential trasparency

An important concept in functional programming is referential transparancy. Referential transparency is a property of programs, that makes it easier to verify, optimize, and parallelize programs.

From Wikipedia's definition:

>**Referential transparency** and **referential opacity** are properties of parts of computer programs. An expression is said to be referentially transparent if it can be replaced with its value without changing the behavior of a program (in other words, yielding a program that has the same effects and output on the same input). The opposite term is referential opaqueness.

A necessary, but not sufficient, condition for referential transparency is the absence of side effects.
In simple words, it's all about the fact that an expression can can be replaced with its value. This obviously requires that the expression (that could be a function call, for example) has no side effects and always return the same output on the same input.

In other words, referential transparency means that the value of expression can depend only on the values of its parts, and not on any other facts about them or about the execution context.

The opposite of referential transparency is referential opacity, where the value does change when evaluated, and the expression cannot be replaced by a
single value without altering the way a program executes.

A trivial but effective example of referential transparency has been introduced in the previous section, with the code snippet about lists. Using immutable lists, the act of adding, removing or updating a value in those data structures results in new lists being created with the changed values, and any part of the program still observing the original lists sees no change.

## Equational reasoning

The previous section introduced and depicted referential transparency. The importance of referential transparency expressions is that they make substitution model work and the substitution model helps **equational reasoning** work.

A possible definition of equational reasoning is *the process of interpreting code by substituting equals-for-equals*.

Equational reasoning is an important property, since it keeps the models easy to reason about and easy to validate. The correctenss of the properties of a model that respect equational reasoning can be verified just as the properties of a math theorem.
