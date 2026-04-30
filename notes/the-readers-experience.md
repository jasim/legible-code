# The Reader's Experience

The purpose of all the structural principles in this guide is to serve the reader. Two properties characterize code that is legible: narrative clarity and low cognitive load.

## Narrative Clarity

A well-structured codebase has the quality of a well-told story. For any given event, there is a *spine* — a traceable path from trigger to outcome — that a reader can follow without needing to hold the entire system in their head. This is the code-level manifestation of Brooks's *conceptual integrity*: the system feels like it was designed by one mind, with a coherent plot.

Narrative clarity doesn't mean everything lives in one file. It means that at each point along the spine, the reader can see:

- What just happened (the data arriving at this stage)
- What will happen next (the transformation or decision at this stage)
- Where the result goes (the output or the next stage)

When narrative structure is absent, the reader must reconstruct the plot from scattered clues — a callback registered here, a state mutation there, an event emitted somewhere else. Each jump consumes a chunk of working memory. After a few jumps, the reader's mental model collapses.

**Remedy:** For every significant event path in the system, ensure there is a single place where the reader can see the *outline* of the story — even if the details live elsewhere. This might be a top-level function that calls each stage in sequence, a pipeline where each step's input and output are visible, or a module whose exports correspond to the stages of the event path. The details can be deep (hidden behind well-named function calls), but the spine must be *shallow* — visible in one reading.

## Cognitive Load

Cognitive load is what the reader must hold in working memory to understand the code at any given point. It accumulates from five sources:

1. **Nested control flow.** Each level of `if/else`, `try/catch`, or loop adds a branch the reader must track. Deeply nested code forces the reader to maintain a mental stack.
2. **Mixed iterations.** A single loop or pipeline processes multiple unrelated pieces of data. The reader must track which transformation applies to which data.
3. **Action at a distance.** A function modifies state that another distant function reads. The reader must hold both locations in mind simultaneously.
4. **Implicit conventions.** The code relies on naming conventions, file placement rules, or framework magic that the reader must know to understand what happens.
5. **Dispersed branching.** A single logical decision is split across multiple locations — part of the condition checked here, part checked there, the else case somewhere else entirely.

**Remedies** (each addressed by a principle elsewhere in this guide):

| Source of load | Remedy |
|---|---|
| Nested control flow | Flatten with early returns and guard clauses; structure as pipelines ([Making Data Explicit](./making-data-explicit.md)) |
| Mixed iterations | Split into separate passes, one concern per iteration ([Cohesion and Decomposition](./cohesion-and-decomposition.md)) |
| Action at a distance | Make data explicit, pass it through the pipeline rather than storing it in ambient state ([Making Data Explicit](./making-data-explicit.md)) |
| Implicit conventions | Make conventions visible as types, signatures, or module structure ([Making Knowledge Explicit](./making-knowledge-explicit.md)) |
| Dispersed branching | Lift invariants into types so the question is removed from downstream code ([Making Evidence Explicit](./making-evidence-explicit.md)) |