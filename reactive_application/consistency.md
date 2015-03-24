# Consistency

In distribuited system, one of the most famous theorem is Eric Brewer's **CAP theorem.**

From Wikipedia:

>In theoretical computer science, the CAP theorem states that it is impossible for a distributed computer system to simultaneously provide all three of the following guarantees:

>- **Consistency** (all nodes see the same data at the same time)
- **Availability** (a guarantee that every request receives a response about whether it succeeded or failed)
- **Partition tolerance** (the system continues to operate despite arbitrary message loss or failure of part of the system)

What the theorem highlights is the impossibility of keeping both consistency and availlability in a distribuited system, since partitions and failures are invevitable.
In other words, during a network partition at least one of consistency and availlability need to be sacrificed.
CAP theorem is often used a justification for using weaker consistency models.

Distributed systems therefore build on a different set of goals called **BASE** instead of the **synchrony-based ACID**. BASE is an acronym for *Basically Available Soft-state services with Eventual-consistency*.

From Wikipedia:
>In summary, the BASE methodology is characterized by high availability for first-tier services, leaving some kind of background cleanup mechanism to resolve any problems created by optimistic actions that later turn out to have violated consistency.

The key point here is the loss of strong consistency, in favor of **eventual consistency**. Eventual consistency may be defined as:
> a consistency model used in distributed computing to achieve high availability that informally guarantees that, if no new updates are made to a given data item, eventually all accesses to that item will return the last updated value.

What eventual consistency means is that when modifying data, the changes need time to travel between each part of the system, and during this time an external observer may see data which are inconsistent, but that eventually will become consistent.

