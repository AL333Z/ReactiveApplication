# Algebraic data types

From Wikipedia:

>In computer programming, particularly functional programming and type theory, an algebraic data type is a kind of composite type, i.e. a type formed by combining other types. Two common classes of algebraic type are product types, i.e. tuples and records, and sum types, also called tagged or disjoint unions or variant types.

The definition introduces some new terminologies: *Sum Type* and *Product Type*.

Wikipedia defines **Sum Type** as:

> a data structure used to hold a value that could take on several different, but fixed types. Only one of the types can be in use at any one time, and a tag field explicitly indicates which one is in use.

An example of a Sum Type in the scala library is Either, that represents a value of one of two possible types. Instances of Either are either an instance of Left or Right. A common use of Either is as an alternative to Option for dealing with possible missing values and convention dictates that Left is used for failure and Right is used for success.

The implementation of Either is something like:

```
sealed abstract class Either[+A, +B]{...}
final case class Left[+A, +B](a: A) extends Either[A, B] {...}
final case class Right[+A, +B](b: B) extends Either[A, B] {...}
```

Basically, Sum Types express an *OR* of types. Remebering that in logic OR is presented by a plus, we can say that algebraically: *type Either = Left + Right*.

Wikipedia defines **Product Type** as:

> another, compounded, type in a structure. The "operands" of the product are types, and the structure of a product type is determined by the fixed order of the operands in the product.

A Product Type is nothing more than a cartesian product of data types. A trivial example may be the following:

```
sealed trait Animal {
    def uniqueId: Long
    def name: String
    def owner: Option[Person]
}

case class Dog(uniqueId: Long, name: String,
    owner: Option[Person], microchipId: String) extends Animal

case class Hamster(uniqueId: Long, name: String,
    owner: Option[Person]) extends Animal

//...
```

In this example, a Dog data type is the collection of all valid combinations of the tuple (Long, String, Option, String). So, algebraically, we can say: *type Dog = Long x String x Option x String*.

Always in this example, we also have that Animal is a Sum Type, since an Animal is a Dog, OR an Hamster, and so on..

Sum Type and Product Type are really important in functional domain modelling, since they provide the abstaction needed for structuring the various data of the domain model.

The values of a Sum Type are typically grouped into several classes, called *variants*. As the name suggests, variants let us model the variations within a specific data type. Each variant has its own constructor, which takes a specified number of arguments with specified types.

Product Types represent a larger abstraction instead, that allows to *clubbing* together some types and *tagging* it with a new data type. This extra tag could also be avoided, simply expressing every Product Type in terms of a tuple of types, but in funcional domain modelling it's always convenient to express an entity with a tagged data type.

Values of algebraic data types are analyzed with **pattern matching**, which helps keep functionality local to the respective variant of algebraic data type. Pattern matching identifies a value by its constructor or field names and extracts the data it contains. Starting from the previous example, a trivial example of pattern matching:

```
  var myAnimal: Animal = ???

  val noise = myAnimal match {
  	case Dog(uniqueId, name, owner, microchipId) => "bau!"
  	case Hamster(uniqueId, name, owner) => "squit!"
  }
```

Algebraic data structures, pattern matching and the power of a strongly typed language like Scala really help in the domain modelling phase.
