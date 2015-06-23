# Architecting an application with MVVM

This section will explore a foundamental aspect of mobile application development: the architectural pattern.

When developping a mobile applications, usually the target platform implicitely or explicitely suggests an architecture.

## State of the art in iOS

In the iOS world, Apple encourages the usage of the Model-View-Controller (MVC) architectural pattern. Quoting the "Start Developing iOS Apps Today" Apple's documentation:

>**MVC** assigns the objects in an app to one of three roles: **model**, **view**, or **controller**.
In this pattern, models *keep track* of your app’s data, views *display* your user interface and *make up the content* of an app, and controllers *manage* your views. By responding to user actions and populating views with content from the data model, controllers serve as a *gateway* for communication between the model and views.

![Traditional MVC](https://developer.apple.com/library/mac/documentation/General/Conceptual/CocoaEncyclopedia/Art/traditional_mvc.gif)

In its original abstraction, in MVC:
- the user manipulates a view and, as a result, an event is generated
- a controller receives the event and manage apply an appliction-specific strategy
- this strategy can consist in requesting a model object to update its state or in requesting a view object to change its appearance.
- the model object notifies all objects who have registered as observers when its state changes; if the observer is a view object, it may update its appearance accordingly.

![Cocoa version of MVC](https://developer.apple.com/library/mac/documentation/General/Conceptual/CocoaEncyclopedia/Art/cocoa_mvc.gif)

However, in an attempt to enhance code reusability, Apple suggests developers to adopt a modified version of MVC, in which there's a strong isolation from models and views, and in which controllers act an intermediary between one or more of an application’s view objects and one or more of its model objects.

Even if views and view controllers are technically distinct components and Apple suggests to keep these decoupled, they are almost always paired. There are a lot reason that cause this trend of strictly coupling a view and its controller:
- the framework provides a big set of components in which a controller already has and manages a view (`UIViewController`, `UITableViewController`, `UICollectionViewController`, `UISplitViewController`, `TabBarController`, ...)
- the UIViewController class usually contains a lot of UI-related code
- storyboards, that enable developers or designers to define GUIs, reason in term of view controllers and "segue" between view controllers

With this said, it is reasonable to modify the previous model with an update in which the view and the view controller are paired togheter.

![A revised iOS application architecture, in which the view and the controller are coupled](http://i61.tinypic.com/2n7ur8.png)

The iOS community, during the years, has adopted (also unconsciously) this architectural pattern to design applications. This has often lead to what is called the "Massive View Controller" anti-pattern. The term "Massive" is used to denote the use and abuse of the view controller entity to host most of the application logic, grouping functionalities that don't stricly relate to the controller entity.

## State of the art in Android

Android's documentation is less explicit about the architectural pattern that developers should use.

At first sight, the overall architecture looks similar at iOS's architecture, with the notion of `Activity` that is similar to `UIViewController` and the notion of `UIView` that is similar to `UIView`.

But more in depth, things are far more complicated. Infact, the main flaw comes from the assumption that there should only be one running `Activity` at a time, and that this `Activity` should be tied down to a single view.
To overcome the limitation of having only a single view tied to an activity, Android introduced the `Fragment` class. This new  abstraction allows an activity to show and manage a certain number of fragments, but still has some limitations like, for example, the possibility of manage stack of fragments.

From this point of view, the iOS platform offers a clearer abstractions, with the notion of viewcontrollers, views and viewcontroller container that allows the developer to express multi-level view hierarchy.

Many developers suggest that what Android is offering is a broken abstraction.

Knowing the limitations of the abstractions that the platform offers, the most popular architectural pattern for Android is a variant of MVC (similar to Apple's variant), in which the `Activity` (and related fragments) is both a the view and the controller.
Google encourages developers to split each view in a fragment, and then each fragment should interact with its parent activity, also to coordinate his action of commands with other fragments.

## Toward a common architecture

On both the platforms, the MVC architectural pattern and its variations doesn't seem to be a perfect fit.

In this section and in the following ones a new architectural pattern will be proposed to better model mobile applications: the **Model-View-ViewModel** pattern.

The reason of the introduction of MVVM in this thesis is that this pattern is an application pattern that isolates the user interface from the underlying business logic, and, as introduced in the previous section, one of the biggest issue of the usage of the MVC pattern is the fact that in both Android and iOS there's not a clear distinction between the view and the controller entities.

The MVVM pattern consists of the following parts:
- the **Model**, which provides a view-independent representation of your business entities.
- the **View**, which is the user interface. It displays information to the user and fires events in response to user interactions.
- the **ViewModel**, which is the bridge between the view and the model. Each View class has at least a corresponding ViewModel class. The ViewModel retrieves data from the Model and manipulates it into the format required by the View. It notifies the View if the underlying data in the model is changed, and it updates the data in the Model in response to UI events from the View.

![The MVVM architecural pattern](https://i-msdn.sec.s-msft.com/dynimg/IC416621.png)

Looking at the diagram, it's clear that the view-model:
- sits between the model and the view, wrapping all the presentation logic
- receives the events and commands from the view
- updates the view once the model has been update

The main reason to use MVVM is the it reduces the complexity of one’s view controllers or activity and makes one’s presentation logic easier to test.

RP frameworks are a perfect fit in MVVM, since they allows to bind views and their associated view-models, allowing the proper synchronization to reflect the changes in both directions.
