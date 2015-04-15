# Lifting

The term *lifting* is used to depict the process of **converting** an ordinary operator to a variant that can operate on behaviors. This process is essential for the end user of the language/framework, since it ensures conciseness and composability.
In other words, lifting is used to move from the non-reactive world to reactive behaviors.

Lifting a **value** creates a **constant behavior** while lifting a **function** **applies the function continuously** to the argument behaviors. There is no similar lifting operation for events, since an event would need an occurrence time as well as a value.

Lifting an operation can be formalized by the following definition, assuming functions that only take one parameter (generalising to functions that take multiple arguments is trivial):

$$
lift: f(T)  \rightarrow f_{lifted} (Behavior < T >).
$$

The $$f_{lifted}$$ function can now be applied to behaviors that holds values of type $$T$$. Lifting enables the language or framework to register a dependency graph in the application's dataflow graph.

Starting from the previous definition, the evaluation of a lifted function called with a behavior at the time step $$i$$ can be formalized as follows:

$$
f_{lifted}(Behavior < T >)  \rightarrow f(T_i) .
$$

where $$T_i$$ is the value of the behavior at the time step $$i$$.

In the literature there are at least three main lifting strategies:

1. **Implicit lifting**, that happens when an ordinary language operator is applied on a behaviour and it is automatically lifted. Implicit lifting is often used by dynamically typed languages. Formally:
$$
f(b1)  \rightarrow f_{lifted}(b1) .
$$

2. **Explicit lifting**, usually used by statically typed langueges. In this case the language provides a set of combinators to lift ordinary operators to operate on behaviors. Formally:
$$
lift(f)(b1)  \rightarrow f_{lifted}(b1) .
$$

3. **Manual lifting**, when the language does not provide lifting operators. Formally:
$$
f(b1)  \rightarrow f(currentValue(b1)) .
$$
