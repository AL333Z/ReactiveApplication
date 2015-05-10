# Signal

As introduced previously, Signal implements the abstraction of **hot signal** as a signal that typically has **no start and no end**.

A typical applicative example for this type can be represented by the press events of a button or the arrive of some notifications. They are just events that happens with no relevance of the fact that someone is observing them.

Using a popular philosophical metaphore:
>If a tree falls in a forest and no one is around to hear it, **it does make a sound**.

In many cases, a stream of events that has no start and no end can't terminate with an error. A simple example of this case is a stream of press events from a button. To overcome this possibility, RAC introduces the type `NoError`, meaning that the signal can't error out.

For the "button example", the type for the related signal might be `Signal<Void, NoError>`, that means:
- the user of the signal only cares about the occurence of the event, no further information will be provided
- the signal can't error out, since in this particular case it has no sense to model the fact that a button press can fail

The button example is just one of many others. If we consider the amount of events that come from the UI, a lot of trivial examples come out quickly:
- a signal that models the changes of a text field
- a signal that models the arrival of notifications (remote, local, etc...)
- any other signal created combining other signals
- ...
