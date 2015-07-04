# On Android

After introducing the use case, the first thing to do is to write down the types involved in the computation and to provide a class that handles the network requests, returning an observable with the response. This class will be part of the network layer of the application, and will expose methods like the following:

```
public Observable<List<Word>> getWords(int month, int year);
```

As a reference, the `Word` type is the following:

```
public class Word {
    public long id;
    public String word;
    public int day;
    public int month;
    public int year;
}
```

The semantics for this observable is pretty simple:

- it *yields* a single results (the response body) or an error (a network error, or a server error, etc..)
- it *starts* the computation each times a consumer *subscribes* itself to the observable

NB: for this kind of request, the behavior would also have been implemented with a future.

Once the network layer is in place, all the transformation needed can be expressed in term of operators.

```
myServiceProvider.getWords(month, year)

    // network operations in io scheduler
    .subscribeOn(Schedulers.io())

    // Observable<List<Word>> -> Observable<Word>
    .flatMap(wordList -> Observable.from(wordList))

    // sort elements
    .toSortedList((l, r) -> { ...sort predicate... })

    // Observable<List<Word>> -> Observable<Word>
    .flatMap(list -> Observable.from(list))

    // build an Observable<Pair<Integer, Word>> (the integer value is the index)
    .map(word -> new Pair<>(0, word))
    .scan((sum, item) -> new Pair<>(sum.first + 1, item.second))

    // build list adapter, highlighting the first element
    .map(indexItemPair -> { // Observable<Pair<Integer, Word>> ->            Observable<WordListItem>
        WordListAdapter.WordListItem item = new WordListAdapter.WordListItem(indexItemPair.second);
        item.setHighlighted(...); // highlight current day item
        return item;
    })
    .toList() // converting to list, since the adapter need a list

    //  UI update on main thread
    .observeOn(AndroidSchedulers.mainThread())

    // subscribing to items, errors, ...
    .subscribe(
        wordItemList -> {
            setListAdapter(new WordListAdapter(wordListActivity, wordItemList));
        },
        throwable -> {
        ...
        }
    );
```

The code shows some transformations and some usage of the most common operators of RxJava.

In the code snippet, `flatMap` is used to transform an `Observable<List<Word>>` to an `Observable<Word>`. This is a pretty common transformation when dealing with list of elements, and since operators like `map`, `filter`, etc.. need to operate on single elements, this operation is essential to build the chain.

Another pattern is the usage of `map` in conjuction with `scan`, to accumulate the result of a computation. The code presented does the trivial job of associating to each element its index in the observable, but it's a use case useful enought to deserve a citation.

The presentation of the items is provided by the `WordListAdapter` and `WordListItem` classes.

One last things to note is the usage of `subscribeOn` and `observeOn`, to keep the main thread responsive.
