# Future and Promises

The chapter about Functional Programming ended with a section that introduced a possible classification of the essential effects in programming.

| | One | Many |
| -- | -- | -- |
| Synchronous | T/Try[T] | Iterable[T] |
| Asynchronous | **Future[T]** | Observable[T] |

This section will introduce a first abstraction to achieve sequential composition of actions that can take time to complete and that can also fail: `Future[T]`.

**Future** and **Promise** are two really powerful abstractions that can be classified in the *asynchronous programming* category.

Wikipedia defines Futures and Promises as follows:

>In computer science, **future**, **promise**, and **delay** refer to constructs used for synchronization in some concurrent programming languages. They describe an object that acts as a proxy for a result that is initially unknown, usually because the computation of its value is yet incomplete. [...]

>A future is a read-only placeholder view of a variable, while a promise is a writable, single assignment container which sets the value of the future.

Futures and Promises promotes an asynchronous and non-blocking approach to programming. The scale up to use multiple cores on a machine, but with the currently implementations they don't scale out across nodes.
