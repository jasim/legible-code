# Growing a Language

## The generative grammar of design

Sussman and Abelson (*SICP*, 1985) identified the three elements every powerful design system provides: **primitives** (the atomic building blocks), **means of combination** (ways to compose smaller things into bigger things), and **means of abstraction** (ways to name compound things so they can be used as if they were primitives). This triad is the generative grammar of software design — the same structure applies whether you're designing a language, a library, or a domain model.

## Growing, not building

Steele ("Growing a Language," OOPSLA 1998) adds the complementary insight: a good language (or domain model) is one that can be *grown* from small pieces. You start with a few well-chosen primitives. Users (or later code) compose those primitives into bigger pieces. Those bigger pieces should be indistinguishable from the primitives — they compose the same way, they're invoked the same way. The key test: "new words defined by a library should look just like the primitives of the language."

## Semantic compression

The result is a *layered vocabulary*:

- **Layer 0:** The fundamental primitives (types, basic operations)
- **Layer 1:** Compositions of primitives into common patterns
- **Layer 2:** Compositions of Layer 1 concepts into higher-level operations
- **Layer N:** The vocabulary at which the application speaks

Each layer is built entirely from the layer below it. Each layer *compresses* the vocabulary of the lower layer — it gives a name to a frequently-occurring pattern, allowing the reader to treat a complex idea as a single chunk (Miller's chunking principle).

This is what Muratori calls **semantic compression** ("Semantic Compression," 2014): the code's vocabulary mirrors the problem domain, with frequently-expressed ideas given their own names and used consistently. Well-compressed code is easy to read because there's minimal code, and the semantics mirror the real "language" of the problem.

Critically: don't compress prematurely. Muratori's rule — "make your code usable before you try to make it reusable" — guards against the abstraction-too-early failure mode. Extract and name only when you've seen the pattern at least twice.

## Remedy

- **Identify the primitives.** What are the smallest, most precise operations and types in your domain? These should be small, pure, and independently understandable.
- **Look for repeated patterns.** When the same combination of primitives appears in multiple places, extract it and name it. But only when you've seen it at least twice.
- **Build upward.** Each successive layer should feel like a natural vocabulary built from the layer below. Reading a high-level function should feel like reading a sentence in the domain language.
- **Ensure composability.** The composed pieces should combine the same way the primitives do. If composing two high-level concepts requires dropping down to the primitive layer, the abstraction has a leak.