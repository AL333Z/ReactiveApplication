# Considerations

The usage of both the library seems to be a good fit for the simple case study introduced.

The imlementation that uses Akka Stream looks a little bit more verbose, since there's some "setup and wiring work" that needs to take place, but the final result looks pretty elegant.

The implementation that uses RxScala is pretty clean and straight-forward, indeed.

For both the solutions, the most important thing to note is the fact that all the computations is build starting from the definition of a static-typed chain of operators (or flows).
