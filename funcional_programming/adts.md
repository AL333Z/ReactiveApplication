# ADTs and Immutability

The previous section introduced ADTs as composed types and depicted Sum Type and Product Type. This section will explain how to achieve referentially trasparency through **immutability**, **avoiding in-place mutations**.

The importance of having immutable data structures is to research in the fact that they simplify the management of parallel and concurrent systems. If data structures are immutables, they can be freely shared between different execution contexts, without any fears.

To better undestand what does it mean to have an immutable data structure, let's consider the following example from the book "Functional Programming in Scala":

```scala
// List data type
sealed trait List[+A]

// data constructors
case object Nil extends List[Nothing]
case class Cons[+A](head: A, tail: List[A]) extends List[A]
```
The operation of addition of an element to the front of the list can be performed **without modify or copy** the object itself, just **reusing** the actual list with a new element at the beginning.

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

This property of reuse object instead of modify or copy the object itself is called *sharing*.

The indroduction of this chapter depicteds three elements as the foundamental domain elements: entities, value objects and services. So, how do ADTs relate to these elements?
As previously written, a value object is *semantically immutable*, so abstracting this with an ADT is a good choice. For example, let's consider the following example:

```scala
case class Address(no: String, street: String, city: String, state: String, zip: String)
```

If an application, at a given time, has an instance of an address and needs to modify an attribute (e.g. the zip code), all it has to do is to **generate a new instance** of the object with the updated attribute, **avoiding in place mutation** (no vars, no setters). Scala also offer a `copy` method to further simplify the job.

```scala
val initialAddress = Address("10", "Via Sacchi", "Cesena", "IT", "47522")
val newAddress = initialAddress.copy(zip = "47523")
```

This approach may looks nice at a first sight, but doesn't scale well, unfortunately. To see the problem, just consider a new entity that uses the address as a value object.

```scala
case class Person(id: Long, name: String, address: Address)
```

The entity Person is *semantically mutable*, but it's implemented with an ADT. The first part of this chapter already stated the fact that an entity is semantically mutable doesn't prevent us from using immutable constructs for its implementation, and this is a brilliant proof of concept.

Back to the issues with when using `copy` for creating a new ADTs with modified fields, let's consider the following simple example.

```scala
val initPerson = Person(0, "Alessandro", initialAddress)
val newPerson = initPerson.copy(address = initPerson.address.copy(zip = "47523"))
```

It immediately appears that the code is getting bad. And things only get worse when there are multiple level of nesting of the objects. A better abstraction to solve this problem is given by *Lenses*. Lenses are ADTs, and in Scala can be implemented as follows:

```scala
case class Lens[O, V](
    get: O => V,
    set: (O, V) => O
)
```

From the implementation it emerges that **Lenses**:
- are **parametrized** (on O and V types)
- have a **getter** method, that takes an object with type O and return a value of type V
- have a **setter** method, that takes an object of type O and returns a new instance of the object set to the value

To demostrate that lenses just work as the copy method introduced below, let's consider the following example:

```scala
val newAddress2 = addressZipLens.set(initialAddress, "47523")
newAddress == newAddress2 //> res0: Boolean = true

```

The compare on the final line asserts that the addresses created with the two different techniques are equals.

To solve the problem related to the multiple level of nesting attribute update it is necessary to introduce a generic `compose` function, implemented as follows:

```scala
def compose[Outer, Inner, Value](
    outer: Lens[Outer, Inner],
    inner: Lens[Inner, Value]
) = Lens[Outer, Value](
    get = outer.get andThen inner.get,
    set = (obj, value) => outer.set(obj, inner.set(outer.get(obj), value)))
```

Compose, as the name suggests, takes care of composing two lenses. Lens composition is really helpful to mantain the code clean and safe, as the following sample will demostrate.

```scala
val personAddressZipLens: Lens[Person,String] = compose(personAddressLens, addressZipLens)
newPerson2 = personAddressZipLens.set(initPerson, "47523")
newPerson == newPerson2 //> res1: Boolean = true
```

As the previous example, also in this case the result is the same as the case when `copy` was used instead. Lenses offer a *view on data*, allowing to get and modify the data *the functional way*.
