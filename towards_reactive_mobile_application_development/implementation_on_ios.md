# Implementation on iOS

This final section will depicts an implementation on the iOS platform for the introduced architecture.

## Model

The `Word` type only contains some read-only properties and a constructor, just like the Android conterpart.

```swift
struct Word {
    let id: Int
    let word: String
    let day: Int
    let month: Int
    let year: Int
    let imageUrl: String

    init(id: Int, word: String, day: Int, month: Int, year: Int, imageUrl: String){
        self.id = id
        self.word = word
        self.day = day
        self.month = month
        self.year = year
        self.imageUrl = imageUrl
    }
}
```

The `WordResponse` struct only wraps an array of `Word`, and also the logic for failed requests (that is omitted for the sake of concisness).

```swift
struct WordResponse {
    let words: [Word]

    init(values: [Word]) {
        words = values
    }
}
```

The `WordService` class only expose a public method that returns an `SignalProducer<WordResponse>`. Also in this case, the real implementation of this method is not important for the purpose of this thesis, and it's omitted.

```swift
class WordService {
    init() {}

    func getWords(month: Int, year: Int) -> SignalProducer<WordResponse, NSError> {
    ...
    }
}
```

## Data binding

The additional data binding layer introduced in Android, in iOS and RAC is already in place, with the `MutableProperty` and `ConstantProperty` types that are already depicted in the section about RAC.

## ViewModel

Also the viewmodel layer is pretty the same as its Android counterpart. All the abstraction used are similar, and even the name are pretty much the same, so there's no need to repeat the description of the code.

The only things that are relevant in this implementation are the types used and the chain of operators.

```swift
class WordListViewModel {

    let isLoading = MutableProperty<Bool>(false)
    let words = MutableProperty<[WordViewModel]>([WordViewModel]())

    private let wordService: WordService

    init(wordService: WordService) {

        self.wordService = wordService

        // retrieve the words, create a view model for each words, and update
        // the overall view model
        self.wordService.getWords(1, year: 2015)
            |> on(started: { self.isLoading.put(true) })
            |> flatMap(FlattenStrategy.Latest, { SignalProducer(values: $0.words) })
            |> map({ WordViewModel(word: $0)})
            |> collect
            |> observeOn(QueueScheduler.mainQueueScheduler)
            |> start(next: {
                wordViewModelList in
                self.isLoading.put(false)
                self.words.put(wordViewModelList)
            }, error: {
                self.isLoading.put(false)
                println("Error \($0)")
            })
    }
}
```

The same arguments are valid for the `WordViewModel` class.

```swift
class WordViewModel {

    let wordTitle: ConstantProperty<String>
    let day: ConstantProperty<Int>
    let month: ConstantProperty<Int>
    let year: ConstantProperty<Int>
    let imageUrl: ConstantProperty<String>

    init (word: Word) {
        self.wordTitle = ConstantProperty(word.word)
        self.day = ConstantProperty(word.day)
        self.month = ConstantProperty(word.month)
        self.year = ConstantProperty(word.year)
        self.imageUrl = ConstantProperty(word.imageUrl)
    }
}
```

## View

The view layer is represented by three main components:
- a `UIViewController`, named `WordsViewController`, that contains a `UITableView`;
- a `UITableViewCell`, named `WordCellView`;
- an helper class, named `TableViewBindingHelper`, that make it easier to bind a table view with a viewmodel;

The entry point of the application is the `WordsViewController` class, that is shown at app launch. The implementation is pretty similar to the implementation of the `MainActivityFragment` of the previous section.

The view controller creates a `WordService`, that then is passed to a `WordListViewModel`.

After these operation, it binds the viewmodel's properties:
- to set the visibility of the load activity indicator;
- to show the list of items, through an helper class.

The last things that the view controller performs is to register its interest in the item selection events from the table view. In this simple use case, the view controller just shows a message in an alert view, but in more real use case scenario this can be the entry point for a presentation of another view controller or other actions.

```swift
class WordsViewController: UIViewController {

    @IBOutlet weak var loadActivityIndicator: UIActivityIndicatorView!
    @IBOutlet weak var wordsTable: UITableView!

    private var bindingHelper: TableViewBindingHelper<WordViewModel>!

    override func viewDidLoad() {
        super.viewDidLoad()

        let wordService = WordService()
        let viewModel = WordListViewModel(wordService: wordService)

        // bind ui to current loading status
        loadActivityIndicator.rac_hidden <~ viewModel.isLoading.producer |> map { !$0 }
        wordsTable.rac_alpha <~ viewModel.isLoading.producer |> map { $0 ? CGFloat(0.5) : CGFloat(1.0) }

        // bind the table view with the view model
        bindingHelper = TableViewBindingHelper(tableView: wordsTable,
            sourceSignal: viewModel.words.producer, nibName: "WordCell")

        // observe selection event and show another view with the selected content
        bindingHelper.getTableViewSelectedItemSignal()
            |> observeOn(UIScheduler())
            |> observe(next: { self.showWordDetail($0) })
    }

    func showWordDetail(wordViewModel: WordViewModel) {
        // simply showing an alert..
        let alert = UIAlertView(title: "Selection", message: "You selected: \(wordViewModel.wordTitle.value)", delegate: nil, cancelButtonTitle: nil, otherButtonTitles: "Ok")
        alert.show()
    }
}
```

The `WordCellView` class represents a single cell in the table view, and use the property of the viewmodel to which is bind to show the values of the current word.

```swift
class WordCellView: UITableViewCell, ReactiveView {

    @IBOutlet weak var yearLabel: UILabel!
    ... // other ui stuff

    func bindViewModel(viewModel: AnyObject) {

        let triggerSignal = self.rac_prepareForReuseSignal.asSignal() |> toVoidSignal

        if let wordViewModel = viewModel as? WordViewModel {
            // bind the text of the labels to its value
            yearLabel.rac_text <~ wordViewModel.year.producer |> map { "\($0)" }
            monthLabel.rac_text <~ wordViewModel.month.producer |> map { "\($0)" }
            dayLabel.rac_text <~ wordViewModel.day.producer |> map { "\($0)" }
            wordLabel.rac_text <~ wordViewModel.wordTitle.producer

            // download the image
            picImageSignalProducer(wordViewModel.imageUrl.value)
                |> startOn(scheduler)
                |> takeUntil(triggerSignal)
                |> observeOn(QueueScheduler.mainQueueScheduler)
                |> on(started: { self.wordImageView.image = nil })
                |> start(next: {
                    self.wordImageView.image = $0
                })
        }
    }

    private func picImageSignalProducer(imageUrl: String) -> SignalProducer<UIImage, NoError> {
        // download and returns the image
    }
}
```

The `TableViewBindingHelper` is an helper class, that save the developer a lot of boilerplate code. The relevant part of the code is the following, which illustrates an initializer that takes a `SignalProducer` of items (viewmodels for the cells of the table view) and a public method that returns a Signal of items, representing the items selected by the user.

```swift
// a helper that makes it easier to bind to UITableView instances
// initial implementation: http://www.scottlogic.com/blog/2014/05/11/reactivecocoa-tableview-binding.html
class TableViewBindingHelper<T: AnyObject> : NSObject, UITableViewDelegate {

    //MARK: Properties
    ...

    //MARK: Public API
    init(tableView: UITableView, sourceSignal: SignalProducer<[T], NoError>, nibName: String) {
        ...
        sourceSignal.start(next: {
            data in
            // reload the table view each time a new array of viewmodels is set
            self.dataSource.data = data.map { $0 as AnyObject }
            self.tableView.reloadData()
        })
    }

    // returns a Signal which emits the items selected by the user
    func getTableViewSelectedItemSignal() -> Signal<T, NoError> {
         ...
    }
}
```
