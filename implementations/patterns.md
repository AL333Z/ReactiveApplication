# Patterns

The previous sections of this chapter introduced the basics of functional domain modelling, by using **algebraic data types**, and the importance of **referential transparency**.

This final section will go deeper, and cover three functional design pattern: **monoids**, **functors** and **monads**. These patterns are really important to fully understand the usage of algebraic data type to model the domain.

The pattern introduced in this section are *abstract*, meaning that they don't operate on just one specific data type bar on all data types that share a common algebra.

## Monoid

**Monoid** is an ubiquitous pattern that come up frequently in programming, even if we are not aware of it. To introduce monoids, let's consider an example about the concatenation operation for List and String types:

- the concatenation operation is a binary operation and is associative
- both List and String have an identity element and the operation applied with the identity yields the same value as the other argument

The points depicted above are a valid algebra for the abstraction of a monoid. In Scala a monoid can be expressed as follows.

```
trait Monoid[A] {
  def op(a1: A, a2: A): A
  def zero: A
}
```

More formally, a monoid needs to satysfy the following laws:

- *Left identity*: `op(zero, a) == a`
- *Right identity*: `op(a, zero) == a`
- *Associativity*: `op(a1, op(a2, a3)) == op(op(a1, a2), a3)`

Back to the initial example, with the given definition List and String concatenation can be expressed as monoids as folllows.

```
val stringMonoid = new Monoid[String] {
  def op(a1: String, a2: String) = a1 + a2
  def zero = ""
}

def listMonoid[A] = new Monoid[List[A]] {
  def op(a1: List[A], a2: List[A]) = a1 ++ a2
  def zero = Nil
}
```

The benefits of having operations that operate on different data types modelled as monoids is given by the fact that monoids offer *parametricity* and the ability to *abstract over behaviors*.

A brilliant example that illustrates the power of monoids is given when they are used in conjuction with lists and list folding functions. Looking at `foldLeft` function definition:

```
def foldLeft[B](z: B)(f: (B, A) => B): B
```

and in the particular case of `A = B`:

```
def foldLeft(z: A)(f: (A, A) => A): A
```

it can be observed that the function signature perfectly matches the signature of `op` operation on the monoid type.

This brings to a really nice proof of concept about the pratical usage of monoids, like in the following example.

```
val words = List("Hello", "world")
val t = words.foldLeft(stringMonoid.zero)(stringMonoid.op)
```

## Functor

The previous section introduced an ubiquitous pattern that recurs frequently in programming, monoid. This section will make a futher step towards the king abstraction of functional programming, that will be introduced in the next one.

In the Scala standard library there are some classes that provide a `map` function. For example:

```
def map[B](f: (A) => B): List[B]

def map[B](f: (A) => B): Option[B]
```

In the classes of the example, the effect of `map` is:

- for List, `map` applies the function `f` on each element of the list
- for Option, `map` aplies the function `f` only if the element is instance of `Some` and return `Some` of the result, otherwise it returns `None`

All this function signatures are pretty the same, with the only difference of the concrete type involved. As always in programming, when there are things that are pretty the same, it's possible that factorize the behavior and create a new abstraction. In this case, the abstraction it's all about a *data type that implements map* and it's called **functor**. In Scala, if a data type `F` implements `map`, `F` is a functor. An implementation of this abstraction can be represented by the following trait.

```
trait Functor[F[_]] {
    def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

**NB**: in the example provided above (from the Scala library), the signatures of the map function are different from the signature of map in the trait introduced below. The reason of this difference is due to the object oriented nature of the classes in the Scala library. The new signature has a more "functional" approach, with an additional parameter (the first one) that takes the object to map on.

With this new abstraction defined, it's possible to define functors explicitely for List and Option in the following way, reusing the object oriented implementation provided by the Scala standard library:

```
def listFunctor: Functor[List] = new Functor[List] {
    def map[A, B](a: List[A])(f: A => B): List[B] = a map f
}

def optionFunctor: Functor[Option] = new Functor[Option] {
    def map[A, B](a: Option[A])(f: A => B): Option[B] = a.map(f).orElse(None)
}
```

Functors are nothing more that an abstraction that has the capability of mapping over some data structure, with functions.

## Monad

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

```
trait Monad[M[_]] extends Functor[M] {
  def unit[A](a: => A): M[A]
  def flatMap[A,B](ma: M[A])(f: A => M[B]): M[B]

  def map[A,B](ma: M[A])(f: A => B): M[B] =
    flatMap(ma)(a => unit(f(a)))
}
```

Monad extends functor, implementing `map`. The trait introduced above is generic, so each type that implements a `unit` and a `flatMap` and respects some laws (depicted below) is a monad.

**NB**: in the literature, `flatMap` is often called `bind` and `unit` is often called `return`.

Two example of monads are represented by `Option` and `List`. Starting from the definition of Monad, they can be defined as follows (reusing the implementation provided by the Scala standard library):

```
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

```
trait M[T] {
    def flatMap[U](f: T => M[U]): M[U]
    def unit[T](x: T): M[T]
}

```
with `map` defined as `m flatMap (x => unit(f(x)))`.

As introduced previously, monads need to satisfy three laws: associativity, left unit and right unit.

###Associativity

The associativity law is about the order of **composition**,  and demands that for any monad `m`and any two functions `f` and `g`, it must not make a difference whether you apply f to m first and then apply g to the result, or if you first apply g to the result of f and then in turn flatMap this over m. Formally:

```
m flatMap f flatMap g == m flatMap (x => f(x) flatMap g)
```

###Left unit

The left unit law demands that flatMap must behave in such a way that for any function f passed to it, the result is the same as calling f in *isolation*. Formally:

```
unit(x) flatMap f  ==  f(x)
```

The left unit law can be considered the most important law, since it guarantes that flatMap let's you apply a transformation to the value contained in a monad, without leaving the monad. In other words, monads promote and allows *containment*  and *chainability* of transformations.

###Right unit

The right unit law demands that applying unit to a monad has no effects at all (or, better, has the same outcame as not calling it at all). Formally:

```
m flatMap unit  ==  m
```

Usually the term monad is also used *informally*, to describe a type that has `flatMap` and `unit`, without any attention about the monad laws.
An example of this is given by the `Try` type, that has a `Success` case that contains a value and a `Failure` case that contains an exceptions.

```
abstract class Try[+T]
case class Success[T](x: T)       extends Try[T]
case class Failure(ex: Exception) extends Try[Nothing]
```

**Try** is often used to wrap the result of a computation inside a container type. The important thing about Try is that is handles exception, through *materialization*.

```
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
