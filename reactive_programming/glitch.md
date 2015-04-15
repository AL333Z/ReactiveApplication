# Glitches

A **glitch** is a **temporary inconsistency** in the observable state. Due to the fact that updates do not happen instantaneously, but instead take time to compute, the values within a system may be **transiently out of sync** during the update process. This may be a problem, depending on how tolerant the application is of occasional stale inconsistent data.
For example, consider a computation that is run before all its dependent expressions are evaluated and that can result in fresh values being combined with stale values.

The paper "A Survey on Reactive Programming" by Bainomugisha et al. depict the problem with the following program:

```code
var1 = 1
var2 = var1 * 1
var3 = var1 + var2
```

![Momentary view of inconsistent program state and recomputation.](http://i61.tinypic.com/2dl5ftd.png)

The image related to the program above shows how the state of the program results incorrect (`var3` is evaluated as 3 instead of 4) and that this also leads to a wasteful recomputation (since `var3` is computed one time more than the necessary).

Glitches only happens when using a *push-based* evaluation model and can be avoided by arranging expressions in a topologically sorted graph. This implementation detail ensures that an expression is always evaluated after all its dependants have been evaluated.
