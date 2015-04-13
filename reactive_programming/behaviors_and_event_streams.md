#Behaviors and Event Streams

Reactive programming introduces two special abstractions:
- **behaviors**, values that are continuos functions of time.
- **event streams**, values which are discrete functions of time.

**Behaviors** are dynamic/evolving values, and are first class values in themselves. You can define them and combine them, pass them into and out of functions. Behaviors are built up out of a few primitives, like constant (static) behaviors and time (like a clock), and then with sequential and parallel combination. n-behaviors are combined by applying an n-ary function (on static values), "point-wise", i.e., continuously over time.

Examples of behaviours are the following:
- time
- a browser window frame
- the cursor position
- the position of a image during an animation
- audio data
- ...

**Events** enable to account for discrete phenomena, and each of which has a stream (finite or infinite) of occurrences. Each occurrence has an associated time and value. Formally, the points at which an event stream is defined are termed *events* and events are said to *occur* on an *event stream*.

Example of event streams are the following:
- timer events
- key presses
- mouse clicks
- MIDI data
- network packets
- ...
