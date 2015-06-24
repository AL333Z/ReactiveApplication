# Considerations

The usage of Rx as a gateway to **bind** the viewmodel to the view and as a wrapper for **computations** that imply **latency** and/or possible **failures** seems to be a good fit, even if the problems related to memory leaks and resource disposal have not been taken care with a particular attention. Thus, the code proposed may have some issues and should be considered only a proof of concept at the moment.

Both the implementation of the proposed architecture seems to be pretty elegant and similar in respect of each other. Also the name of each component is consistent over the platforms, where possible.

One could also think to create a *meta-meta-model* that allows to describe the architecture of the application with some high-level DSL and then generate the code for both the platforms, but this argument goes outside the scope of this thesis.
