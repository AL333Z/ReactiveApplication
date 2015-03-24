# Referential trasparency

From Wikipedia:

> Absence of side effects is a necessary, but not sufficient, condition for referential transparency. Referential transparency means that an expression (such as a function call) can be replaced with its value; this requires that the expression has no side effects and is pure (always returns the same results on the same input).






The concept of referential transparency is that replacing an entire expression with a single value will have no impact on the execution of the program. A more concrete way to think about it is that performing an operation on a particular datum or data set will result in a new value being returned, with no impact on the original data upon which you performed the operation, and no side effects will occur. When using an immutable list, the act of adding, removing or updating a value in that data structure should result in a new list being created with the changed values, and any part of the program still observing the original list sees no change.

Consider Java’s java.lang.StringBuffer class. If we were to call the reverse method on a StringBuffer instance, we will have a new instance of the StringBuffer with the values reversed. However, the original StringBuffer instance was also changed by making this call.

```java
StringBuffer myStringBuffer = new StringBuffer(“foo”);
StringBuffer myNewStringBuffer = myStringBuffer.reverse();
System.out.println(String.format(
    "myStringBuffer: %s, new value: %s",
    myStringBuffer,
    myNewStringBuffer));

// Result - myStringBuffer: oof, new value: oof
```

This is an example of referential opacity, where the value upon which you performed the operation does change when evaluated, and the expression cannot be replaced by a single value without altering the way a program executes. It’s worth noting that the same issues exists for the java.lang.StringBuilder class. In functional programming, referential transparency is required to reason about our application at runtime.
