# A use case

To illustrate the application of the MVVM pattern in conjuction with RP frameworks, let's introduce a simple-but-effective use case.

The use case is an app for iOS and Android, that will:
- fetch a web service to request a list of words;
- each words has a set of attributes, such as: an id, a title, a day, a month and a year which is related to, and an url to an image;
- after querying the web service, the result should be showed in a list. Each list item should display all the attributes, with also the image;
- a detail view should be opened when the user taps on an item.

The requirements are pretty trivial for an experienced developer that use to work following the "standard" way to develop mobile applications, but in this thesis author's hopinion is a pretty significative example, since it allows to demostrate a lot of concepts:
- abstracting the retrieval, manipulation and presentation of data (also see the previous dedicated section);
- presenting an uniform abstraction, applyable on both the platforms;
- proper separation of concerns, applying MVVM.
