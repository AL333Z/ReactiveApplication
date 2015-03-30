# Monad

The concept of monad comes from category theory, and is one of the most important topic in functional programming development.

From Wikipedia:

>In functional programming, a **monad** is a structure that represents computations defined as sequences of steps: a type with a monad structure defines what it means to chain operations, or nest functions of that type together. This allows the programmer to build pipelines that process data in steps, in which each action is decorated with additional processing rules provided by the monad. As such, monads have been described as "programmable semicolons"; a semicolon is the operator used to chain together individual statements in many imperative programming languages, thus the expression implies that extra code will be executed between the statements in the pipeline.

From a Erik Meijer's quote:

>Monads are return types that guide you through the happy path.

From a Martin Odersky's quote:

>Monads are parametric types with two operations flatMap and unit that obey some algebraic laws.

An other definition can be:
>Monads are abstract computations that help us mimic the effects of typically impure actions like exceptions, IO, continuations etc. while providing a functional interface to the users.

All this quotes and definitions help to give us the idea of what a monad is.
Formally, a monad is just a type, and can be defined as follows:

```scala
trait Monad[M[_]] extends Functor[M] {
  def unit[A](a: => A): M[A]
  def flatMap[A,B](ma: M[A])(f: A => M[B]): M[B]

  def map[A,B](ma: M[A])(f: A => B): M[B] =
    flatMap(ma)(a => unit(f(a)))
}
```

Monad extends functor, implementing `map`. The trait introduced above is generic, so each type that implements a `unit` and a `flatMap` and respects some laws (depicted below) is a monad.

**NB**: in the literature, `flatMap` is often called `bind` and `unit` is often called `return`.

Two example of monads are represented by `Option` and `List`. Starting from the implementation of Monad, they can be defined as follows.

```scala
val optionMonad = new Monad[Option] {
    def unit[A](a: => A) = Some(a)
    def flatMap[A,B](ma: Option[A])(f: A => Option[B]) = ma flatMap f
}

val listMonad = new Monad[List] {
    def unit[A](a: => A) = List(a)
    def flatMap[A,B](ma: List[A])(f: A => List[B]) = ma flatMap f
}
```

From the example, it's easy to note that `unit` is different for each monad. For example, for the `List` type unit is `List(a)` and for the `Option` type unit is `Some(a)`.

A more object oriented definition for monad is the following.

```scala
trait M[T] {
    def flatMap[U](f: T => M[U]): M[U]
    def unit[T](x: T): M[T]
}

```
with `map` defined as `m flatMap (x => unit(f(x)))`.

As introduced previously, monads need to satisfy three laws: associativity, left unit and right unit.

##Associativity
The associativity law is about the order of **composition**,  and demands that for any monad `m`and any two functions `f` and `g`, it must not make a difference whether you apply f to m first and then apply g to the result, or if you first apply g to the result of f and then in turn flatMap this over m. Formally:

```scala
m flatMap f flatMap g == m flatMap (x => f(x) flatMap g)
```

##Left unit
The left unit law demands that flatMap must behave in such a way that for any function f passed to it, the result is the same as calling f in *isolation*. Formally:

```scala
unit(x) flatMap f  ==  f(x)
```

The left unit law can be considered the most important law, since it guarantes that flatMap let's you apply a transformation to the value contained in a monad, without leaving the monad. In other words, monads promote and allows *containment*  and *chainability* of transformations.

##Right unit
The right unit law demands that applying unit to a monad has no effects at all (or, better, has the same outcame as not calling it at all). Formally:

```scala
m flatMap unit  ==  m
```

Usually the term monad is also used *informally*, to describe a type that has `flatMap` and `unit`, without any attention about the monad laws.
An example of this is given by the `Try` type, that has a `Success` case that contains a value and a `Failure` case that contains an exceptions.

```scala
abstract class Try[+T]
case class Success[T](x: T)       extends Try[T]
case class Failure(ex: Exception) extends Try[Nothing]
```

**Try** is often used to wrap the result of a computation inside a container type. The important thing about Try is that is handles exception, through *materialization*.

```scala
def map[S](f: T => 􏰀S): Try[S] = this match {
  case Success(value)       => Try(f(value))
  case failure: Failure(t)  => failure
}


def flatMap[S](f: T => 􏰀Try[S]): Try[S] = this match {
  case Success(value)       =>
    try { f(value) } catch { case NonFatal(t) => Failure(t) }
  case failure: Failure(t)  => failure
}
```

At first sight Try looks like a monad, implementing `flatMap` (and `unit`), but formally it doesn't respect the left unit law.
