# Monoid

**Monoid** is an ubiquitous pattern that come up frequently in programming, even if we are not aware of it. To introduce monoids, let's consider an example about the concatenation operation for List and String types:
- the concatenation operation is a binary operation and is associative
- both List and String have an identity element and the operation applied with the identity yields the same value as the other argument

The points depicted above are a valid algebra of the abstraction of a monoid. In Scala a monoid can be expressed as follows.

```scala
trait Monoid[A] {
  def op(a1: A, a2: A): A
  def zero: A
}
```

More formally, a monoid needs to satysfy the following laws:
- Left identity: `op(zero, a) == a`
- Right identity: `op(a, zero) == a`
- Associativity: `op(a1, op(a2, a3)) == op(op(a1, a2), a3)`

Back to the initial example, with the given definition List and String concatenation can be expressed as monoids as folllows.

```scala
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

```scala
def foldLeft[B](z: B)(f: (B, A) => B): B
```

and in the particular case of `A = B`:

```scala
def foldLeft(z: A)(f: (A, A) => A): A
```

it can be observed that the function signature perfectly matches the signature of `op` operation on the monoid type.

This brings to a really nice proof of concept about the pratical usage of monoids, like in the following example.

```scala
val words = List("Hello", "world")
val t = words.foldLeft(stringMonoid.zero)(stringMonoid.op)
```
