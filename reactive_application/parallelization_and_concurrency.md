# Latency, concurrency, parallelism

In the previous section some core concepts have been introduced. This section will go deeper and will cover how a Reactive Application can actually *react to users, timely*. Reacting to users basically (and quite obviously) means that an application should *respond to requests it receives*.

In distribuited systems **latency is a fact of life**. Their design implicitly hide latency where each component interact with each others.

A mean to put latency down is represented by *concurrency* (and *parallelism*).
Concurrency stuff is well-researched, with literature increasingly available.

From the Wikipedia's definition of **concurrency**:
>In computer science, concurrency is a property of systems in which several computations are executing simultaneously, and potentially interacting with each other. The computations may be executing on multiple cores in the same chip, preemptively time-shared threads on the same processor, or executed on physically separated processors. A number of mathematical models have been developed for general concurrent computation including Petri nets, process calculi, the Parallel Random Access Machine model, the Actor model and the Reo Coordination Language.

From the Wikipedia's definition of **parallel computing**:
>Parallel computing is a form of computation in which many calculations are carried out simultaneously, operating on the principle that large problems can often be divided into smaller ones, which are then solved concurrently ("in parallel").
[...] Parallel computer programs are more difficult to write than sequential ones, because concurrency introduces several new classes of potential software bugs, of which race conditions are the most common. Communication and synchronization between the different subtasks are typically some of the greatest obstacles to getting good parallel program performance.

In many cases, if for the completion of a generic request several other services must be involved, then **the overall result will be obtained quicker if the other services can perform their functions in parallel**.

Parallelizing computations generally brings some form of *interaction* (and problems to solve/prevent), such as *cooperation*, *competition/contention* and *interferences*.

*Amdahl's law* is often used in parallel computing to predict the theoretical maximum speedup using multiple processors, and says:
>The speedup of a program using multiple processors in parallel computing is limited by the time needed for the sequential fraction of the program.

Formally, the maximum speedup parallelizing a system:

$$S = {1 \over {1 - P + {P \over N} }}.$$

Where:
- `P` is the portion of a program that can be made parallel
- `(1 - P)` is the part that cannot be parallelized
- `N` is the number of processors

![The speedup of a program using multiple processors in parallel computing is limited by the sequential fraction of the program. For example, if 95% of the program can be parallelized, the theoretical maximum speedup using parallel computing would be 20× as shown in the diagram, no matter how many processors are used.](http://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/AmdahlsLaw.svg/648px-AmdahlsLaw.svg.png)

From the graph emerges that serializations and sequentializations of jobs are bad in term of overall performances.

Another variable to keep in mind is *efficiency*, that is a normalized measure of speed-up indicating how effectively each processor is used. Formally:

$$ E = {S \over P}. $$

One of the biggest issue when designing distribuited systems is represented by their capability of *partial failure*, which means that in addition to a failure within the target service it has to be considered the possibility that, for example, either the request or the response might be lost. In order to recognize that a request has failed it may be introduced the concept of *bounded latency*, which means formulating a dependable budget for which latency is allowed.

Let's now consider a system that receives requests faster than it can handle them. As an example, immagine a service that can process 100 request per second and which receives 110 requests per second for some time. After the first second, 10 request are still waiting to be processed (under the hypothesis that there's a queue somewhere..), and this will obviously leads to an increasing delay in processing requests and will also be perceived by the users, since the delay will keep rising, unless the incoming arrival of requests slows down.
These concepts are nothing new to telecommunications engineers, which have already solved these problems years ago (see Erlang's laws and related literature).

**Within the service**, the latency may be controlled by introducing **bounded queues**, that essentially trasport information from one execution contect to another. Queues introduce a delay withing the system that is proportional to the current queue size (a.k.a. system load). These queues are explicit and deliberately placed inside the system. An important aspect to keep in mind when tuning queues is the size of each queue. Infact, from the size of a queue depends the latency and usually it is set in a way that *it holds only as many elements as extra latency is allowed by the system*. While the queue is full and new requests arrive, they just fail, immediately.

![Representation of a FIFO (first in, first out) queue, from Wikipedia](http://upload.wikimedia.org/wikipedia/commons/thumb/5/52/Data_Queue.svg/300px-Data_Queue.svg.png)

A further technique to reduce latency is *shielding users from overwhelmed services*. If a service is struggling under an heavy load (with an increasing latency and, consequently, failed requests) it is a good idea to immediatly insulate users from it. This will also give the chance to the server to recover and empty its queues. Ideally, this concept may be formalized with the notion of **circuit breakers** from electrical engineering. Circuit breakers enable to codify failure strategies to address failures of external systems in advance.

![A circuit breaker representation. Image from a Typesafe's presentantion.](https://dl.dropboxusercontent.com/u/5045423/circuit-breaker.png)
