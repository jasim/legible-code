# Sources

## Functional Core, Imperative Shell

Gary Bernhardt — ["Boundaries"](https://www.destroyallsoftware.com/talks/boundaries) (RubyConf, 2012).

All decisions, validations, and transformations happen in a *pure core* — functions that take data in and return data out, with no side effects. The *shell* is a thin layer that gathers input from the world, passes it to the core, and mechanically applies the core's output back to the world.

## Defunctionalization

John Reynolds — "Definitional Interpreters for Higher-Order Programming Languages" (1972; reprinted in *Higher-Order and Symbolic Computation*, 1998).

Reynolds's technique replaces higher-order functions with a finite data type whose constructors name each shape the behaviour can take, plus an interpreter that consumes the data type and produces the original effect. The canonical bridge between "code that does things" and "data that describes what should happen."

## Reification

Smith — *Reflection and Semantics in a Procedural Language* (MIT, 1982). (I haven't read this paper myself, but reification has become a very useful tool in our collective vocabulary that I've taken it for granted for a long time.)   

To reify is to make implicit machinery explicit *as data* — to take a process, decision, or piece of structure that lives only in the runtime and turn it into a value that can be inspected, stored, transformed, and replayed. Reification is the structural move that makes "data has primacy over actions" possible.

## Primitives, Combination, Abstraction

Gerald Jay Sussman & Hal Abelson — *Structure and Interpretation of Computer Programs* (MIT Press, 1985).

Every powerful design system provides three things: primitives, means of combination, and means of abstraction. This is the generative grammar of software design.

## Why Functional Programming Matters

John Hughes — ["Why Functional Programming Matters"](https://www.cse.chalmers.se/~rjmh/Papers/whyfp.html) (1990).

The value of functional programming lies in its *glue*. Higher-order functions and lazy evaluation let small, simple components compose into larger programs without the seams mattering. The clearest early defence of pipelines and composition as primary design tools.

## Growing a Language

Guy Steele — ["Growing a Language"](https://www.cs.virginia.edu/~evans/cs655/readings/steele.pdf) (OOPSLA, 1998).

A good language — or domain model — is one that can be *grown* from small pieces. Start with well-chosen primitives. Compose them into bigger pieces. The bigger pieces should be indistinguishable from the primitives.

## DRY (Don't Repeat Yourself)

Andrew Hunt & Dave Thomas — *The Pragmatic Programmer* (1999).

"Every piece of knowledge must have a single, unambiguous, authoritative representation within a system." Often misread as "don't write similar-looking code" — the point is about *knowledge*, not *characters*.

## Simple Made Easy

Rich Hickey — ["Simple Made Easy"](https://www.infoq.com/presentations/Simple-Made-Easy/) (Strange Loop, 2011).

**Simple** is an objective, structural property: one role, one task, one concept, one dimension — not interleaved with anything else. Its opposite is **complex**, from Latin *complecti*, "to braid together." **Easy** is subjective: near at hand, familiar — and easy things are often deeply complected underneath. Most legibility failures are places where independent concerns have been braided together.

## Boolean Blindness

Robert Harper — ["Boolean Blindness"](https://existentialtype.wordpress.com/2011/03/15/boolean-blindness/) (2011); popularized for working programmers by Alexis King in "Parse, Don't Validate" (2019).

A `boolean` discards the evidence that produced it. The remedy is to return a richer type that *carries* the evidence, so the type system enforces what was learned.

## Make Illegal States Unrepresentable

Yaron Minsky and the Jane Street OCaml tradition — ["Effective ML"](https://blog.janestreet.com/effective-ml-video/) talks and posts (~2011); *Real World OCaml* (Minsky, Madhavapeddy, Hickey, 2013).

Design types so that invalid combinations of values cannot be constructed. If the state can't exist, you don't have to handle it.

## Parse, Don't Validate

Alexis King — ["Parse, Don't Validate"](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) (2019).

Validation *checks* that data is well-formed but throws away the evidence. Parsing *converts* data into a richer type where invalid states are unrepresentable. The shell does the parsing; the core only ever sees the parsed form.

## The Value of Values

Rich Hickey — ["The Value of Values"](https://www.infoq.com/presentations/Value-Values/) (JaxConf, 2012).

Values are immutable, comparable by structure, and carry no implicit context. Programs become easier to reason about as more of the system is expressed in values rather than places.

## Semantic Compression

Casey Muratori — ["Semantic Compression"](https://caseymuratori.com/blog_0015) (2014).

Treat programming like dictionary compression: when you see the same pattern twice, extract it and name it. Don't compress prematurely — "make your code usable before you try to make it reusable."

## Deep Modules

John Ousterhout — *A Philosophy of Software Design* (2018).

A deep module provides powerful functionality behind a simple interface. A *shallow* module has a complex interface relative to the functionality it provides. Shallow modules push complexity onto their callers; deep modules absorb it.

## Information Hiding

David Parnas — "On the Criteria to Be Used in Decomposing Systems into Modules" (1972).

The right criterion for module boundaries is not "what step does this perform?" but "what design decision does this hide?"

## Conceptual Integrity

Fred Brooks — *The Mythical Man-Month* (1975).

The system should feel like it was designed by one mind, with a coherent plot. Conceptual integrity is the most important consideration in system design.

## Domain Modeling Made Functional

Scott Wlaschin — *Domain Modeling Made Functional* (2018) and the [F# for Fun and Profit](https://fsharpforfunandprofit.com/) essays.

A practical synthesis of type-driven design: discriminated unions for domain choices, smart constructors for invariants, total functions, and "make illegal states unrepresentable."

## Type-Centric Modularization

Yaron Minsky and the Jane Street tradition; the broader OCaml/ML community. See *Real World OCaml* (2013) and the standard ML library convention.

Every domain concept gets its own module, named after the type it defines. The type *is* the organizing principle: the module boundary follows from the type, not from a process step or a feature grouping.

## Chunking

George A. Miller — "The Magical Number Seven, Plus or Minus Two" (1956).

Working memory can hold roughly 7 ± 2 items. *Chunking* — grouping related items into a single named unit — effectively expands working memory by allowing the reader to treat a complex idea as one item rather than many.
