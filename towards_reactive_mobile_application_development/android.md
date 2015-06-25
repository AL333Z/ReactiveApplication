# Implementation on Android

The previous section proposed an overall architecture for the use case. This sections will depict a possible implementation for the Android platform.

## Model

Starting from the **model**, things are pretty straightforward.

The `Word` type only contains some read-only properties and a constructor.

```java
public class Word {

    public final long id;
    public final String word;
    public final int day;
    public final int month;
    public final int year;
    public final String imageUrl;

    public Word(int id, String word, int day, int month, int year, String imageUrl) {
        this.id = id;
        this.word = word;
        this.day = day;
        this.month = month;
        this.year = year;
        this.imageUrl = imageUrl;
    }
}
```

The `WordResponse` class only wraps an array of `Word`, and also the logic for failed requests (that is omitted for the sake of concisness).

```java
public class WordResponse {
    final public Word[] words;
    WordResponse(Word[] words) {
        this.words = words;
    }
}
```

Finally, the `WordService` class only expose a public method that returns an `Observable<WordResponse>`. Also in this case, the real implementation of this method is not important for the purpose of this thesis, and it's omitted. The real important thing of this class is the return type of the method signature.

```java
public class WordService {
    public WordService() { ... }
    public Observable<WordResponse> getWords(int month, int year) {
        ...
    }
}
```

## Data binding

To better understand the following steps, it's necessary to introduce the abstraction of `MutableProperty` and `ConstantProperty`. On the iOS conterpart, ReactiveCocoa already offers the corresponding abstraction. RxJava and RxAndroid don't directly offer the conceptual equivalent, but it's pretty easy to build the same abstraction starting from a `BehaviorSubject`.

```java
public interface Val<T> {
    T value();

    boolean hasValue();

    Observable<T> observe();
}
```

The `Val` interface introduces the notion of a **value** of type `T`.

```java
public interface Var<T> extends Val<T> {
    void put(T value);
}
```

The `Var` interface introduces a **variable** of a type `T`.

```java
public class MutableProperty<T> implements Var<T> {
    private final BehaviorSubject<T> subject;
    private Val<T> val;

    protected MutableProperty() {
        subject = BehaviorSubject.create();
    }

    protected MutableProperty(T defaultValue) {
        subject = BehaviorSubject.create(defaultValue);
    }

    public static <T> MutableProperty<T> create() {
        return new MutableProperty<>();
    }

    public static <T> MutableProperty<T> create(T defaultValue) {
        return new MutableProperty<>(defaultValue);
    }

    @Override public void put(T value) {
        subject.onNext(value);
    }

    @Override public synchronized Val<T> asVal() {
        if (val == null) {
            val = ConstantProperty.of(this);
        }

        return val;
    }

    @Override public T value() {
        return subject.getValue();
    }

    @Override public boolean hasValue() {
        return subject.hasValue();
    }

    @Override public Observable<T> observe() {
        return subject.asObservable();
    }
}
```

The `MutableProperty` implements the `Var` interface, and represents the abstraction of a property that is **updated over the time**. This kind of property has a notion of current value (that can also be absent) and can be observed, returning an `Observable`.

```java
public class ConstantProperty<T> implements Val<T> {
    private final Var<T> var;

    protected ConstantProperty(Var<T> var) {
        this.var = var;
    }

    public static <T> ConstantProperty<T> of(Var<T> var) {
        return new ConstantProperty<>(var);
    }

    public static <T> ConstantProperty<T> create(T val) {
        return ConstantProperty.of(new MutableProperty(val));
    }

    @Override public T value() {
        return var.value();
    }

    @Override public boolean hasValue() {
        return var.hasValue();
    }

    @Override public Observable<T> observe() {
        return var.observe();
    }
}
```

The `ConstantProperty` implement a property that can have a single-assignment (constant) value.

## ViewModel

Moving to the viewmodel layer, things starts to become interesting.

```java
public class WordListViewModel {

    private static final String TAG = WordListViewModel.class.getSimpleName();

    public final MutableProperty<Boolean> isLoading = MutableProperty.create(false);
    public final MutableProperty<List<WordViewModel>> words = MutableProperty.create(new LinkedList<>());

    public WordListViewModel(WordService wordService) {

        wordService.getWords(1, 2015)
                .doOnSubscribe(() -> this.isLoading.put(true))
                .observeOn(AndroidSchedulers.mainThread())
                .flatMap(wordResponse -> Observable.from(wordResponse.words))
                .map(word -> new WordViewModel(word))
                .toList()
                .subscribe(wordViewModelList -> {
                            this.isLoading.put(false);
                            this.words.put(wordViewModelList);
                        },
                        throwable -> {
                            this.isLoading.put(false);
                            Log.e(TAG, throwable.getMessage());
                        });
    }
}
```

As previously introduced in the architecture diagram, the `WordListViewModel` class has two public `MutableProperties` and exposes a constructor that receives a `WordService`.

When an instance of `WordListViewModel` is created, the viewmodel provides to build the chain of computations needed to perform its job. In this case, it starts a fetch request using the `WordService` instance, and then setting all the proper side-effects.

If everything completes fine, the two mutable properties are updated as follows:
- in `isLoading` is put `false`, so any view that observe that property will be notified of the termination of the request
- in `words` is put a List<WordViewModel>

```java
public class WordViewModel {

    final public ConstantProperty<String> wordTitle;
    final public ConstantProperty<Integer> day;
    final public ConstantProperty<Integer> month;
    final public ConstantProperty<Integer> year;
    final public ConstantProperty<String> imageUrl;

    public WordViewModel(Word word) {
        this.wordTitle = ConstantProperty.create(word.word);
        this.day = ConstantProperty.create(word.day);
        this.month = ConstantProperty.create(word.month);
        this.year = ConstantProperty.create(word.year);
        this.imageUrl = ConstantProperty.create(word.imageUrl);
    }
}
```

The `WordViewModel` class is even simpler, since it only contains a set of `ConstantProperty`, abstracting the set of attributes of the `Word` class.

## View

The view layer is represented by three main components:
- an `Activity`, named `MainActivity`
- a `Fragment`, named `MainActivityFragment`, that is contained in the `MainActivity`
- a RecyclerView.Adapter, named `WordListAdapter`, that is necessary to implement a list view in Android

The `MainActivity`'s only job is to instatiate a `MainActivityFragment`, so its code is omitted.

```java
public class MainActivityFragment extends Fragment {
    private RecyclerView mRecyclerView;
    private ProgressBar mProgressBar;
    private WordListAdapter mAdapter;

    ...

    @Override public void onResume() {
        super.onResume();

        if (mRecyclerView.getAdapter() != null) return; // the adapter has already been set
        final WordService wordService = new WordService();
        final WordListViewModel wordListViewModel = new WordListViewModel(wordService);

        // bind ui to current loading status
        wordListViewModel.isLoading.observe()
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(ViewActions.setVisibility(mProgressBar));

        // bind the adapter view with the view model
        mAdapter = new WordListAdapter(wordListViewModel.words.observe());
        mRecyclerView.setAdapter(mAdapter);

        // observe selection event and show another view with the selected content
        mAdapter.getSelectedWordViewModelObservable()
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(selectedWordViewModel ->
                        Toast.makeText(getActivity(), selectedWordViewModel.wordTitle.value(), Toast.LENGTH_SHORT).show());
    }
}
```

`MainActivityFragment` is a really crucial point of the application. In this class is created a `WordService`, that is then passed to a newly created `WordListViewModel`.

The fragment then binds the viewmodel's properties:
- to set the visibility of a progress bar;
- to show the list of items, through an adapter.

The last things that the fragment performs is to register its interest in the item selection events from the adapter. In this simple use case, the fragment just shows a message in a `Toast`, but in more real use case scenario this can be the entry point for a fragment transaction or for the launch of a new activity.

```java
public class WordListAdapter extends RecyclerView.Adapter<WordListAdapter.WordViewHolder> {
    private List<WordViewModel> mViewModels = new LinkedList<>();
    private PublishSubject<WordViewModel> mAdapterSelectedItemSubject;

    /**
     * Constructor that returns a WordListAdapter, with an observable containing a list of view models
     *
     * @param viewModelsObservable an observable with the list of view models
     */
    public WordListAdapter(Observable<List<WordViewModel>> viewModelsObservable) {
        super();

        viewModelsObservable.subscribe(next -> updateItems(next));
        mAdapterSelectedItemSubject = PublishSubject.create();
    }

    public void updateItems(List<WordViewModel> items) {
        mViewModels.removeAll(mViewModels);
        mViewModels.addAll(items);
        notifyDataSetChanged();
    }

    public Observable<WordViewModel> getSelectedWordViewModelObservable() {
        return mAdapterSelectedItemSubject.asObservable();
    }

    @Override
    public WordViewHolder onCreateViewHolder(ViewGroup viewGroup, int i) {
        View view = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.word_list_item, viewGroup, false);
        return new WordViewHolder(view);
    }

    @Override
    public int getItemCount() { return mViewModels.size(); }

    @Override
    public void onBindViewHolder(WordViewHolder holder, int i) {
        WordViewModel item = mViewModels.get(i);
        holder.bindItem(item);
    }

    @Override
    public void onViewRecycled(WordViewHolder holder) {
        holder.viewRecycledBehavior.onNext(null);
        holder.viewRecycledBehavior = BehaviorSubject.create();
    }

    /**
     * ViewHolder for the item
     */
    class WordViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
        private WordViewModel mItem;

        private TextView mDayTextView; private TextView mMonthTextView;
        private TextView mYearTextView; private TextView mWordTextView;
        private ImageView mImageView;

        BehaviorSubject<Void> viewRecycledBehavior;

        public WordViewHolder(View itemView) {
            super(itemView);
            itemView.setOnClickListener(this);
            ... // bind ui elements to fields
        }

        public void bindItem(WordViewModel item) {
            mItem = item;

            mItem.day.observe().map(d -> d.toString()).subscribe(ViewActions.setText(mDayTextView));
            mItem.month.observe().map(m -> m.toString()).subscribe(ViewActions.setText(mMonthTextView));
            mItem.year.observe().map(y -> y.toString()).subscribe(ViewActions.setText(mYearTextView));
            mItem.wordTitle.observe().subscribe(ViewActions.setText(mWordTextView));

            viewRecycledBehavior = BehaviorSubject.create();

            mImageView.setImageDrawable(null);
            item.imageUrl.observe()
                    .takeUntil(viewRecycledBehavior.asObservable())
                    .subscribeOn(Schedulers.io())
                    .map(url -> downloadImage(url))
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe(drawable -> mImageView.setImageDrawable(drawable));
        }

        private Drawable downloadImage(String imageUrl) {...}

        @Override public void onClick(View v) {
            mAdapterSelectedItemSubject.onNext(mItem);
        }
    }
}
```

The class `WordListAdapter` contains a lot of boilerplate code that directly depend on the undelyind abstractions.
The relevant code is in the `ViewHolder` class, in which:
- a `WordViewModel` is used to retrieve the properties that describe a `Word` model class;
- the `onClick` events are pushed through a `PublishSubject<WordViewModel>`. This subject is then exposed as an `Observable<WordViewModel>`, and will inform each subscriber when the user selects an item of the list;
- given the url of the image, the image is downloaded and showed without blocking the main thread.
