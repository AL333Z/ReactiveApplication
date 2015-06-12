# Abstracting the retrieval, manipulation and presentation of data

The first use case proposed is about a quite common set of actions, such as the **retrieval, manipulation and presentation of some sort of data**.

Every simple or complex application has at least a part in the app lifecycle in which it queries some provider (a cache, a local database, a Rest API) to fetch some resource.

The abstraction of event streams can be used to model this use case in a pretty straigh-forward way.

In the case of a web request, the stream will either emit one value -  containing the body of the response - and succed, or fail with an error.

In the case of more complex request-response configuration (e.g. a request that opens a web socket, that then emits and push new data over a long time) the stream will emit more values, as long as the flow of items continues, or terminate if a failure occurs.

Abstracting the reception of items is only the first part of the scenario introduced. Once the application got some kind of data, a certain number of processing stages can be run over these data. Thus, the notion of event streams and operators fits really nicely also on this part.

To demostrate a possible solution of this scenario using an approach based on RP, lets introduce a sample use case:

> The application has to query a web service, that returns a list of words for a given month. Each word refers to a specific day, month and year. The application should show all the words in the given month, sorted by date, with the first one highlighted with a different color.
