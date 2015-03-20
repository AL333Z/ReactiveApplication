# Event orientation

The previous section highlighted the importance to have isolated and loose coupled components, to achieve resilience.
This section will depict another crucial principle in Reactive Applications that form another piece of the puzzle: **event orientation**.

A system made by many smaller components usually needs the introduction of a *communication protocol*, at least. In Reactive Applications this protocol is *explicit* and is represented by **messages**, which are sent and received by every component of the system. The effect of the usage of  protocols based on message-passing is that developing independently executing compartments will also leads to have largely independent components in the source code.

"Reactive Design Patterns" book clearly depicts the benefits of such an architecture:
>- Components can be **tested** and **verified** in isolation.
- Their **implementation can be evolved without affecting their users**.
- New functionality corresponds to new communication protocols, which are added in a conscious fashion.
- Overall maintenance cost is lower.

In the section *Latency, concurrency, parallelism* it has been mentioned Amdahl's law when depicting the benefits of concurrency and parallelism when dealing with computations that can be divided in parts, executed simultaneously.

Amdahl's law also show us another crucial issue in programming concurrent applications. In particular, it imposes **scalability limitations**.

![The speedup of a program using multiple processors in parallel computing is limited by the sequential fraction of the program. For example, if 95% of the program can be parallelized, the theoretical maximum speedup using parallel computing would be 20× as shown in the diagram, no matter how many processors are used.](http://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/AmdahlsLaw.svg/648px-AmdahlsLaw.svg.png)

If for example the 95% of the program can be parallelized, the theoretical maximum speedup using parallel computing would be 20×, no matter how many processors are used. Putting these quantities in perspective and considering the resource used to reach those results, this is not a huge gain for application performances. The gain (and scalability) is limited by all the synchronization mechanisms introduced to support the communication between each part of the computation.

To remove this limit, Reactive Applications introduces the concept of **shared nothing architecture**. From Wikipedia:
> A shared nothing architecture (SN) is a distributed computing architecture in which each node is independent and self-sufficient, and there is no single point of contention across the system. More specifically, none of the nodes share memory or disk storage.
[...]
Shared nothing is popular for web development because of its scalability. As Google has demonstrated, a pure SN system can scale almost infinitely simply by adding nodes in the form of inexpensive computers, since there is no single bottleneck to slow the system down. Google calls this sharding.


