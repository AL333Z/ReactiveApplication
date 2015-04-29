# Functor

The previous section introduced an ubiquitous pattern that recurs frequently in programming, monoid. This section will make a futher step towards the king abstraction of functional programming, that will be introduced in the next one.

In the Scala standard library there are some classes that provide a `map` function. For example:

```scala
def map[B](f: (A) => B): List[B]

def map[B](f: (A) => B): Option[B]
```

In the classes of the example, the effect of `map` is:
- for List, `map` applies the function `f` on each element of the list
- for Option, `map` aplies the function `f` only if the element is instance of `Some` and return `Some` of the result, otherwise it returns `None`

All this function signatures are pretty the same, with the only difference of the concrete type involved. As always in programming, when there are things that are pretty the same, it's possible that factorize the behavior and create a new abstraction. In this case, the abstraction it's all about a *data type that implements map* and it's called **functor**. In Scala, if a data type `F` implements `map`, `F` is a functor. An implementation of this abstraction can be represented by the following trait.

```scala
trait Functor[F[_]] {
    def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

**NB**: in the example provided above (from the Scala library), the signatures of the map function are different from the signature of map in the trait introduced below. The reason of this difference is due to the object oriented nature of the classes in the Scala library. The new signature has a more "functional" approach, with an additional parameter (the first one) that takes the object to map on.

With this new abstraction defined, it's possible to define functors explicitely for List and Option in the following way, reusing the object oriented implementation provided by the Scala standard library:

```scala
def listFunctor: Functor[List] = new Functor[List] {
    def map[A, B](a: List[A])(f: A => B): List[B] = a map f
}

def optionFunctor: Functor[Option] = new Functor[Option] {
    def map[A, B](a: Option[A])(f: A => B): Option[B] = a.map(f).orElse(None)
}
```

Functors are nothing more that an abstraction that has the capability of mapping over some data structure, with functions.
