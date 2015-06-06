# Graph

Since not every computation is or can be expressed as a linear processing stage pipeline, Akka Streams also provide a graph-resembling DSL for building stream processing graphs, in which each node can has multiple inputs and outputs.

The documentation refers to graph operation as **junctions**, in which multiple flows are connected at a single point, enabling to perform any kind of *fan-in* or *fan-out*.

The Flow graph APIs provide a pretty straight forward abstraction:
- **Flow**s represent the **connection** within the computation graph
- **Junction**s represent the **fan-in and fan-out** point to which the flows are connected

The APIs already provide some of the most useful juctions, like the following:

- `Merge[In]` - *(N inputs , 1 output)* picks randomly from inputs pushing them one by one to its output
- `Zip[A,B]` – *(2 inputs, 1 output)* is a ZipWith specialised to zipping input streams of A and B into an (A,B) tuple stream
- `Concat[A]` – *(2 inputs, 1 output)* concatenates two streams (first consume one, then the second one)
- `Merge[In]` – *(N inputs , 1 output)* picks randomly from inputs pushing them one by one to its output
- `Broadcast[T]` – *(1 input, N outputs)* given an input element emits to each output

The documentation also provide a simple but brilliant example that illustrates how the DSL provided by the library can be used to express graph computation keeping a great level of declarativeness and code readability.

The following image shows a graph that express a computation in which:
- **the edges are flows**
- **the nodes are a sink, a source and two junctions**

![A hand written graph expressing a computation](http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-M2/_images/simple-graph-example.png)

The corresponding computation can be implemented as follows:

```scala
val g = FlowGraph.closed() { implicit builder: FlowGraph.Builder[Unit] =>
  import FlowGraph.Implicits._
  val in = Source(1 to 10)
  val out = Sink.ignore
  val bcast = builder.add(Broadcast[Int](2))
  val merge = builder.add(Merge[Int](2))
  val f1, f2, f3, f4 = Flow[Int].map(_ + 10)
  in ~> f1 ~> bcast ~> f2 ~> merge ~> f3 ~> out
              bcast ~> f4 ~> merge
}
```

When building and connecting each component, the compiler will check for type correctness and this is a really usefull things. The check to control whether or not all elements have been properly connected is done at run-time, though.

The framework also provide the notion of partial graph. A **partial graph** is a graph with *undefined sources, sinks or both*, and it's useful to structure the code in different components, that will be then connected with other components. In other words, the usage of partial graphs favours code **composability**.

In many cases it's also possible to expose a complex graph as a simpler structure, such as a Source, Sink or Flow, since these concepts can be viewed as special cases of a partially connected graph:
- a source is a partial flow graph with exactly one output
- a sink is a partial flow graph with exactly one input
- a Flow is a partial flow graph with exactly one input and exactly one output

One last feature that this section will depict and that Akka Stream supports is the possibility to insert **cycles** in flow graphs. This feature is potentially dangerous, since it may lead to deadlock or liveness issues.

The problems quickly arise when there're unbalanced feedback loops in the graph. Since Akka Stream is based on processing items in a bounded manner, if a cycle has an unbounded number of items (for example, when items always get reinjected in the cycle), the back-pressure will deadlock the graph very quickly.

A possible strategy to avoid deadlocks in presence of **unbalanced cycles** is introducing  a **dropping element on the feedback arc**, that will drop items when back-pressure begins to act.

A brilliant example from the documentation is the following, where a `buffer( )` is used with a 10 items capacity and a `dropHead` strategy.

```scala
FlowGraph.closed() { implicit b =>
  import FlowGraph.Implicits._
  val merge = b.add(Merge[Int](2))
  val bcast = b.add(Broadcast[Int](2))

￼  source ~> merge ~> Flow[Int].map { s => println(s); s } ~> bcast ~> Sink.ignore
      merge <~ Flow[Int].buffer(10, OverflowStrategy.dropHead) <~ bcast
}
```

An alternative approach in solving the problem is by **building a cycle that is balanced from the beginning**, by using **junctions that balance the inputs**. Thus, the previous example can also be solved in the following manner, with:
- a `ZipWith( )` junction, that will balance the feedback loop with the source
- a `Concat( )` combined with another `Source( )` with an initial element that performs an initial "kick-off". Infact, using a balancing operator to balance a feedback loops require an initial element in the feedback loop, otherwise we fall in the "chicken-and-egg" problem.

```scala
FlowGraph.closed() { implicit b =>
  import FlowGraph.Implicits._
  val zip = b.add(ZipWith((left: Int, right: Int) => left))
  val bcast = b.add(Broadcast[Int](2))
  val concat = b.add(Concat[Int]())
  val start = Source.single(0)

  source ~> zip.in0
  zip.out.map { s => println(s); s } ~> bcast ~> Sink.ignore
  zip.in1 <~ concat <~ start
             concat         <~          bcast
}
```
