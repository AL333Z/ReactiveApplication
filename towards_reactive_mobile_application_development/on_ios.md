# On iOS

On iOS, the problem can be solved using the same conceptual abstractions.

Starting with a network provider that expose the call to the APIs with the following method signature:

```swift
func getWords(month: Int, year: Int) -> SignalProducer<[Word], NSError>
```

As a reference, the type `Word` is defined as follows:

```swift
public struct Word {

    public let id: Int
    public let word: String
    public let day: Int
    public let month: Int
    public let year: Int

    init(id: Int, word: String, day: Int, month: Int, year: Int){
        self.id = id
        self.word = word
        self.day = day
        self.month = month
        self.year = year
    }
}
```

Also on iOS, the semantics for this signal producer is pretty simple and similar to the androd counterpart:

- it *yields* a single results (the response body) or an error (a network error, or a server error, etc..)
- it *starts* the computation each times a consumer *subscribes* itself on the signal producer

```swift
getWords(month, year)
    // sort
    |> map { words in words.sorted(...) }

    // SignalProducer<[Word]> -> SignalProducer<Word>
    |> flatMap(FlattenStrategy.Merge, { words in SignalProducer(values: words) })

    // build an SignalProducer<(Int, Word)> (the integer value is the index)
    |> map { word in (0, word) }
    |> scan( (0, nil), { (last, current) in (last.0 + 1, current.1) })

    // observe on UI thread
    |> observeOn(UIScheduler())

    // start and subscribe to elements, errors, ...
    |> start(next: { elem in self.updateView(elem)})
```

Almost every step of the chain is the same as its android conunterpart.
The main difference is that RAC doesn't offer a method to sort the elements of a SignalProducer, so, in this case, the elements are filtered at the beginning of the chain using a method of the `Array` type.
