# A common architecture

Starting from the requirements and with MVVM in mind, what follows is the proposed overall system architecure.

![The overall architecture of the system](http://i62.tinypic.com/24gta28.png)

NB: the diagram is incomplete, and shows only the relevant building block of the system. The convention used here to express the abstraction of stream of items is the RxJava's Observable type.

What immediately emerges looking at the digram is how the MVVM pattern is applied to model the requirements.

The **Model** provides a view-independent representation of the business entities. In this case, it's pretty trivial to express the model of the Word class.

The **View** doesn't show something relevant at this stage, since it heavily depends on platform-specific abstractions, and for this reason it will be better depicted in the next two sections.

The **ViewModel** is the bridge between the view and the model, wrapping all the presentation logic. In particular, there're two concrete type of viewmodels:
- `WordListViewModel`, that represent the viewmodel for the list view, as a whole;
- `WordViewModel`, that represent the viewmodel for a single item of the list.

`WordListViewModel` use a `WordService`, that is a class that wraps all the network and parsing operations, returning a `WordResponse`, that contains an array of `Word`. `WordListViewModel` has two `MutableProperty`:
- `isLoading`, that indicates if there's a fetch request running
- `words`, that contains the update list of words

`WordViewModel` is a pretty simple entity, only containing some `ConstantProperty`, referring to the word attributes. The number of instances of `WordViewModel` will be equal to the number of `Word` returned by the `WordService`.

`MutableProperty` and `ConstantProperty` are two abstractions that help in binding the viewmodel to the view, and allows to build the automatic update of the view layer once the model gets updated.

Note that this architecture is platform-agnostic and there're no reference to a specific platform or native abstractions, except for the `Observable` type (which can be easily traduced in `SignalProducer`, in this specific context).
