# Making Data Explicit

The most important thing to see in a codebase is not the control flow (which function calls which) but the *data flow* (what data enters, how it transforms, what data exits). Control flow is the skeleton; data flow is the blood.

## Data over actions

The principle: every interaction should be immediately converted into a piece of data, and that data should travel visibly through the system, being transformed at each stage, until it reaches an edge where I/O occurs.

This is the philosophical core of functional programming, but it doesn't require a functional language. It requires a discipline: make data explicit. Name intermediate values. Use types to describe the shape of data at each stage. Let the reader see the data *flowing*, not just the functions *calling*.

**Values over places.** Values are immutable, comparable by structure, and carry no implicit context — a date, a number, a tuple, a map. *Places* (variables, fields, references) are mutable holders. Programs become easier to reason about as more of the system is expressed in values rather than places: values can be passed across boundaries, cached, compared, persisted, sent over the wire, replayed in tests. None of this is true of stateful objects. (Hickey, "The Value of Values.")

**Reification.** To reify is to make implicit machinery explicit as data — to take a process, decision, or piece of structure that lives only in the runtime and turn it into a value that can be inspected, stored, transformed, and replayed. A query plan, an AST, a routing table, a state machine: each is a reification of behavior that could otherwise have been buried in imperative control flow. Reification is the structural move that makes "data has primacy over actions" possible.

## Data mapping pipelines

When data must undergo multi-step transformations, structure them as **pipelines** — sequential, declarative stages where each step takes a typed input, produces a typed output, and the intermediate values are named and visible.

Rather than one large block that intermixes multiple transformations on different pieces of data, break it into discrete stages: `rawInput → parsed → enriched → validated → command`. Each arrow is a named function. Each intermediate value has a type. The reader can understand each stage independently, and the pipeline as a whole reads as a sentence: "we take raw input, parse it, enrich it with context, validate business rules, and produce a command."

Named pipeline stages serve as **cognitive rest stops**. When a value undergoes multi-step transformation, each named intermediate lets the reader put down what they were holding and pick up the next stage fresh — working memory is freed between stages instead of accumulating across them.

This is the clearest practical expression of Hughes's argument ("Why Functional Programming Matters," 1990): the value of functional programming lies in its *glue*. Higher-order functions and function composition let small, simple components combine into larger programs without the seams mattering — each stage is written independently against typed inputs and outputs, and the pipeline is assembled by composing them. Pipelines and composition are primary design tools; linear data flow is preferred over interleaved control.

## Remedy

- Convert events and interactions into data structures as early as possible
- Structure transformations as pipelines with named intermediate types at each stage
- Name intermediate data at each transformation stage (don't nest transformations deeply)
- Prefer returning values over mutating state
- Use types to make the shape of data at each stage explicit
- When data must be accessed from multiple places, pass it explicitly rather than storing it in ambient state