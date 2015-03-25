# ADTs and Immutability

The previous section introduced ADTs as composed types and depicted Sum Type and Product Type. This section will explain how to achieve referentially trasparency through immutability, avoiding in-place mutations.

The importance of having immutable data structures is to research in the fact that they simplify the management of parallel and concurrent systems. If data structures are immutables, they can be freely shared between different execution contexts, without any fears.

To better undestand what does it mean to have an immutable data structure, let's consider the following example from the book "Functional Programming in Scala":

```scala
// List data type
sealed trait List[+A]

// data constructors
case object Nil extends List[Nothing]
case class Cons[+A](head: A, tail: List[A]) extends List[A]
```
The operation of adding an element to the front of the list may be performed without modifying or copying the object itself, just reusing the actual list with a new element at the beginning.

```scala
  val initialList:List[Int] = Cons(1, Cons(2, Nil))
  val myList = Cons(0, initialList)
```

This approach can be obviously used also to remove an element from the list (in the following example, the 1 to the front).

```scala
  val initialList:List[Int] = Cons(1, Cons(2, Nil))

  val myList1 = initialList match {
    case Cons(1, xs) => xs
    case l: List[Int] => l
  }
```
This property is called *sharing*.

The indroduction of this chapter depicteds three elements as the foundamental domain elements: entities, value objects and services. So, how do ADTs relate to these elements?



