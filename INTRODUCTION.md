# What is Legible Code?

Legibility in code is fundamentally about narrative clarity. A readable system possesses a visible spine: its steps sit statically together, arranged in a logical sequence. I should be able to read the code and understand its intent without having to mentally simulate the runtime.

To achieve this, we should reduce the code to pure computation. Strip away state mutation and focus on deterministic expressions: logic, arithmetic, and data transformation. The archetype is the classical expression evaluator found in ML-style languages:

```
fn eval_expr(ast: Expr) -> Int {
  match ast {
    Const(value) => value
    Add(left, right) => eval_expr(left) + eval_expr(right)
    Mul(left, right) => eval_expr(left) * eval_expr(right)
    // exhaustive pattern matching statically prevents all other types
  }
}
```

For me, the most habitable code hews as close to this functional ideal as possible.

To get here, we have to sequester all side-effects to the system's boundaries. We construct strict parsers through which all external data must pass before entering the core. They map incoming data into precise domain types, failing loudly on parse errors. The core can thus blindly trust those types, and avoid littering defensive code everywhere.

This reliance on types must extend to our domain logic as well. A function that validates data should not return a boolean; instead it should act as a parser that encodes the validity into the type itself. Downstream consumers can then use that data, without needing to relitigate their properties. This principle — Parse, Don't Validate — operates at the system boundary, where raw input is converted into well-typed domain values. But it also operates inside the pipeline: as data is enriched, sorted, or joined, the invariants that emerge should be lifted into types. An array that has been sorted carries its sortedness in its type; a record that has been enriched carries its enrichment. After lifting, every downstream conditional that branched on the question becomes deletable.

But all this is for naught if the codebase duplicates knowledge. We often see this in codebases where invariants, transformations, and behaviors are scattered. The domain model is fragmented and disorganized. In a legible system, every piece of knowledge has a single, authoritative source. These definitions form a rich, composable vocabulary of pure functions, allowing us to grow a language tailored specifically to the problem at hand.

A legible codebase emphasizes the flow of data. Instead of tangled imperative machinery where logic is interspersed with actions, we get linear data pipelines. They show us a sequence of clear transformations, where intermediate values are given explicit type names. This unbraiding of logic lets us track data as it morphs from raw input to refined output. Named pipeline stages serve as cognitive rest stops: the reader can put down what they were holding and pick up the next stage fresh, instead of accumulating working memory across the entire transformation.

Data must have primacy over actions. Consider a database query execution plan. By reifying complex decisions into a data structure, we create a plan that can be paused, inspected, and optimized. If those decisions were buried in imperative logic, such manipulation would be impossible. Similarly, in our own code, we should separate planning from execution. By breaking complex actions into a data-based plan, we gain a pure functional surface where we can test our decision-making logic independent of execution. The plan itself becomes a readable artifact — it is the narrative of what should happen, made inspectable. A separate, mechanical interpreter then executes it. Intent is no longer implicit in a chain of function calls; it is explicit in a data structure.

This data-centric approach should also dictate our file and module organization. We organize modules around data types, not actions. A major type owns its module, surrounded by the code that constructs, validates, and transforms it. The module boundary follows from the type, not from a process step or a feature grouping. A type creates a gravitational center: operations that primarily concern this type belong here; operations that primarily concern a different type belong there. When facing intractable code, the answer is almost never more functions — it is more types. Extracting functions without introducing types is segmentation, not modularization. The types you identify become your module boundaries.

And we must be careful not to decompose prematurely. When we split a concept, we may lose information that only existed in the relationship between the parts. Downstream code is then forced to reconstruct the original intent from fragments — often through fragile heuristics that pattern-match their way back to information that was already available before the split. In a legible codebase, data carries its full semantic content through the system. We split at consumption points, not at origin. When a specific consumer only needs part of a concept, that consumer extracts what it needs — the source doesn't pre-split on its behalf.

We also design our modules to be deep. The right criterion for a module boundary is not "what step does this perform?" but "what design decision does this hide?" A deep module provides powerful functionality behind a simple interface. It absorbs complexity on behalf of its callers. A shallow module — one whose interface is as complex as its implementation — pushes complexity outward rather than containing it.

And finally, in an ideal codebase, all decisions are made with absolute clarity; there is no need for fragile heuristics, and we design our types so that data never gets into impossible shapes.

All of the above ideas comes down to one thing: **make the invisible visible**. At every scale — expression, function, module, system —  the move is the same: take something hidden in time, space, or logic, and make it a static, inspectable, typed thing. Data hidden in control flow becomes named values flowing through pipelines. From there, knowledge that was scattered across files consolidates into a single authoritative definition. The evidence validation once discarded now returns as a guarantee carried in the type, and intent buried in interleaved logic rises to the surface as a visible plan. Shallow boundaries deepen into modules that hide design decisions, while unnamed repetition grows into a layered language that mirrors the domain.

A codebase that follows these principles reads like a well-surveyed landscape seen from a ridge — every feature lies where you'd expect it, each boundary drawn with purpose, nothing hidden behind a fold in the terrain.
