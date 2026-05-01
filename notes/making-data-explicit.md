# Making Data Explicit

The most important thing to see in a codebase is not the control flow (which function calls which) but the *data flow* (what data enters, how it transforms, what data exits). Control flow is the skeleton; data flow is the blood.

## Data over actions

The principle: every interaction should be immediately converted into a piece of data, and that data should travel visibly through the system, being transformed at each stage, until it reaches an edge where I/O occurs.

## Data mapping pipelines

When data must undergo multi-step transformations, structure them as **pipelines** — sequential, declarative stages where each step takes a typed input, produces a typed output, and the intermediate values are named and visible.

Rather than one large block that intermixes multiple transformations on different pieces of data, break it into discrete stages: `rawInput → parsed → enriched → validated → command`. Each arrow is a named function, and the intermediate values they produce have a well-named type. The reader can understand each stage independently, and the pipeline as a whole reads as a sentence: "we take raw input, parse it, enrich it with context, validate business rules, and produce a command."

Named pipeline stages serve as **cognitive rest stops**. When a value undergoes multi-step transformation, each named intermediate lets the reader put down what they were holding and pick up the next stage fresh — working memory is freed between stages instead of accumulating across them.


## Remedy

- Convert events and interactions into data structures as early as possible
- Structure transformations as pipelines with named intermediate types at each stage
- Name intermediate data at each transformation stage (don't nest transformations deeply)
- Prefer returning values over mutating state
- Use types to make the shape of data at each stage explicit
- When data must be accessed from multiple places, pass it explicitly rather than storing it in ambient state
