# Reactive application

In recent years a lot of changes are happening in the IT world. The main causes of these changes are to be found in how **application requirements** and **user expectations** evolved.

A few years ago a large application had several servers, seconds of response time, hours of offline maintenance and Gigabytes of data. Nowadays, a large applications should offer a response to users in milliseconds with 100% uptime. Data is measured in Petabytes and applications may be deployed almost everywhere, from mobile devices to clusters running an high number of multi-core processors.

|  | Response time | Availlability | Data | Deployment |
| -- | -- | -- | -- | -- |
| Past | seconds | hours of downtime | Gigabytes | Several servers |
| Now | milliseconds | 100% uptime expected | Petabytes | Several multi-core processors, potentially grouped in clusters |

The [Reactive Manifesto](http://www.reactivemanifesto.org) by Typesafe identifies 4 main aspects that take part in designing a Reactive System:
* **Responsive**, meaning that it must react to its users timely ensuring a consistent quality of service.
* **Resilient**, meaning that it must react to failures and ensure availlability even after a failure. This aspect is achieved through replication, containment, isolation and delegation.
* **Elastic**, meaning that it must react to variable load pressure, avoiding contention on shared resources and ensuring scalability.
* **Message Driven**, meaning that it must react to inputs, abstracted in the form of messages.The message driven nature ensure loose coupling, isolation, location transparency, and provides the means to delegate errors as messages.

![](../_book/gitbook/images/reactive-traits.svg)

The Reactive Manifesto captures insights learned by the software development community in building internet-scale systems.

From a Jonas Bonér (Founder & CTO of Typesafe) quote:
>Reactive Systems are all about principles, core fundamental CS principles that have been known to work for years. It is just that they are more valid and important today than ever, and sadly most people are rediscovering them over and over again.

From the abstract of *Principles of Reactive Programming* course of [Coursera](https://www.coursera.org):
>Reactive programming is an emerging discipline which combines concurrency and event-based and asynchronous systems. It is essential for writing any kind of web-service or distributed system and is also at the core of many high-performance concurrent systems. Reactive programming can be seen as a natural extension of higher-order functional programming to concurrent systems that deal with distributed state by coordinating and orchestrating asynchronous data streams exchanged by actors.

In this chapter we will explore some of the main techniques that enable an application to be considered *reactive*.
