#Behaviors and Event Streams

FRP introduces two special abstractions:
- **behaviors or signals**, values that are continuos functions of time.
- **event streams**, values which are discrete functions of time.

**Behaviors** are dynamic/evolving values, and are first class values in themselves. Behaviors can be defined and *combined*, and *passed* into and out of functions.

Behaviors are built up out of a few primitives, like constant (static) behaviors and time (like a clock), and then with sequential and parallel combination. n-behaviors are combined by applying an n-ary function (on static values), "point-wise", i.e., continuously over time.

In other terms, a Behavior simply **sends values over time** until it either **completes**, or **errors out**, at which point it stops sending values forever.

Examples of behaviours are the following:
- time
- a browser window frame
- the cursor position
- the position of a image during an animation
- audio data
- ...

**Events** enable to account for discrete phenomena, and each of it has a stream (finite or infinite) of occurrences. Each occurrence is a value paired with a time. Events are considered *to be improving list of occurences*.

Formally, the points at which an event stream is defined are termed *events* and events are said to *occur* on an *event stream*.

Example of event streams are the following:
- timer events
- key presses
- mouse clicks
- MIDI data
- network packets
- ...

Every instance of FRP conceptually includes both behaviors and events. The classic instance of FRP takes behaviors and events as first-class values.
