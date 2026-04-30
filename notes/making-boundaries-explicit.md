# Making Boundaries Explicit: Deep Modules

## What should a module hide?

Parnas (1972) showed that the right criterion for module boundaries is not "what step does this perform?" but "what design decision does this hide?" A module should encapsulate one thing that might change — a data format, an algorithm, a policy — so that when it does change, only that module is affected.

## Depth: interface simplicity vs. implementation complexity

Ousterhout (*A Philosophy of Software Design*, 2018) extends this with the *deep module* principle: the best modules provide powerful functionality behind simple interfaces. If a module's interface is as complex as its implementation, it's *shallow* — it pushes complexity onto its callers. If a module's interface is simple but hides significant complexity, it's *deep* — it absorbs complexity on behalf of its callers.

Constantine's cohesion/coupling lens gives you the diagnostic: high cohesion within (everything in the module belongs together) and low coupling between (modules don't depend on each other's internals).

## Diagnostics

For each module, ask:

1. **What does this module hide?** If you can't name a single design decision it encapsulates, the boundary is wrong.
2. **Is the interface simpler than the implementation?** If the interface is as complex as what's behind it, the module is too shallow. Consider merging it with an adjacent module to create a deeper one.
3. **Would a change to this module's internals propagate to its callers?** If yes, the interface is leaking implementation details. Redesign the interface around what callers actually need to know.