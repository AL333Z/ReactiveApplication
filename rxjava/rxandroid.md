# RxAndroid

The previous sections cover all the main topic of RxJava. This sections will go a step further, introducing some additional features that bring RxJava to the Android ecosystem.

**RxAndroid** is a separate module of RxJava, that gives some useful bindings to the developer.

The `AndroidSchedulers` package provides some specific scheduler for the *Android threading system*.

The additional schedulers provided are:
- `AndroidSchedulers.mainThread( )`, that will execute an action on the main **Android UI thread**
- `AndroidSchedulers.handlerThread(Handler handler)`, that **uses the provided Handler** to execute an action

A typical example of the usage of `mainThread( )` is the following, that performs a download in the scheduler `io( )` and the show the image to the user:

```java
service.getImage(url)
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(bitmap -> myImageView.setImageBitmap(bitmap));
```

**ViewObservable** is another feature that adds some bindings for android `View` that returns observables of events that come from the UI, such as:
- `clicks( )`, that emits a new item each time a `View` is **clicked**
- `text( )`, that emits a new item each time a `TextView`'s **text content** is changed
- `input( )`, same as above, for `CompoundButton`s
- `itemClicks( )`, same as above, for `AdapterView`s

These methods are useful to bind the events from the user of an application, reifying its action and reacting with some operation, in a declarative way.
