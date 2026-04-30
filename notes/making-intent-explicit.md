# Making Intent Explicit: Plans Over Actions

When logic and I/O are interleaved, the reader cannot see what the code *intends* to do without tracing through every side effect. The remedy: separate the "what should happen" from the "make it happen."

## Functional Core, Imperative Shell

All decisions, validations, and transformations should happen in a **pure core** — functions that take data in and return data out, with no side effects. The **shell** is a thin, "stupid" layer that gathers input from the world, passes it to the core, and then mechanically applies the core's output to the world (database writes, API calls, UI updates).

The core is trivially testable — it's just data in, data out. The shell is so simple it barely needs tests.

## The Interpreter Pattern: Return a Plan, Then Run the Plan

A stronger version: the core doesn't just compute results, it builds a *description* of what should happen — an AST, a recipe, a plan. A separate interpreter then executes the plan. The description is pure and composable; the interpreter is mechanical.

This makes **intent visible as data**. When a user interaction triggers a complex sequence of effects, the plan object *is* the narrative — you can inspect it, log it, test it, and trace it. Instead of intent being implicit in a chain of function calls, it's explicit in a data structure that says "here is what should happen, and in what order." The interpreter is then so mechanical that it barely needs attention.

**Defunctionalization** (Reynolds, 1972) is the theoretical foundation: replace higher-order functions with a finite data type whose constructors name each shape the behavior can take, plus an interpreter that consumes the data type and produces the original effect. Every decision the runtime could make becomes *visible*: listed exhaustively in a sum type, dispatched by a single match. This is the structural ancestor of plan/execute patterns, free monads, and reified query plans.

## Remedy

For any function that mixes logic and I/O:

1. **Extract the decision.** Move the computation into a pure function that takes all necessary data as arguments and returns a result (or a plan).
2. **Push I/O to the edges.** The caller gathers the inputs (database reads, API calls), passes them to the pure function, then applies the result.
3. **For complex multi-effect sequences, return a plan.** When the core needs to describe multiple effects in a specific order, have it return a data structure describing the effects. A separate, mechanical interpreter executes the plan. The "what" (the plan) is separate from the "how" (the execution), and the plan itself is a readable artifact.
4. **Test the core directly.** The pure function can be tested with plain data — no mocks, no setup. If the core returns a plan, test that the plan contains the right steps.

The boundary between core and shell should align with your module boundaries. The core is the "what should happen" module; the shell is the "make it happen" module.