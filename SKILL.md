---
name: legible-code
description: >
  Diagnose and remedy code legibility problems — scattered narrative, invisible data flow,
  tangled I/O, missing domain vocabulary, wrong module seams, cognitive overload, and scattered
  knowledge. Use when reviewing code, planning a refactor, designing a new module, or when the
  user describes code as "tangled," "hard to follow," "hard to test," "hard to change," or asks
  where to split or merge a system. Triggers on PR review, refactor planning, module/seam
  design discussions, and any question about why a piece of code is hard to read or modify. Do
  NOT trigger for greenfield "write me a function" tasks, bug fixes with a clear local cause,
  formatting/lint, or pure performance work.
license: MIT
metadata:
  author: Jasim A Basheer
  version: "2.1.0"
---

# Legible Code

Read each diagnostic below. Note which ones fire for the code in question. Then read the matching remedy. Lower-numbered diagnostics are foundations; higher-numbered ones emerge from them. D12 (Scattered Knowledge) is cross-cutting.

For the reasoning behind these diagnostics, see `OVERVIEW.md`.

## Diagnostics

### D1. Narrative Incoherence
Look at your entry points — HTTP handlers, CLI commands, event subscribers, message consumers, job triggers. Any place where a user action or system event enters the codebase. Pick one entry point. Trace what happens from the moment it receives input to the final side-effect. Note how many files you pass through and whether each step is explicit or implicit.

**You have this problem if:**
- Understanding one event requires jumping across many distant files
- The path goes through implicit callbacks, event emitters, or ambient state mutations
- You can't point to a single place that shows the "outline" of what happens
- A new team member would have to open 5+ files to understand one request

**Remedy:** Read `notes/making-intent-explicit.md` and apply its guidance.

### D2. Invisible Data
As you trace the event path from D1, try to see the *data* at each stage. What shape does it have? What was added? What was transformed?

**You have this problem if:**
- State is mutated in place — a function modifies an object and returns `void`
- Data is accessed through ambient globals, context objects, or "reach-up" patterns
- Data is reconstructed from scattered sources rather than passed explicitly
- You can't describe the type/shape of the data at a given point without reading surrounding code

**Remedy:** Read `notes/making-data-explicit.md` and apply its guidance.

### D3. Tangled Intent and Execution
Look at any function that makes a decision or performs a calculation. Does it also read from a database, call an API, or write to the filesystem? Can you exercise the decision logic without standing up the world around it?

**You have this problem if:**
- You can't test the logic without setting up external services or writing mocks
- A single function both *decides what to do* and *does it* (reads DB, computes, writes DB)
- Business rules are buried inside I/O-heavy functions
- A user interaction triggers an untraceable web of immediate side effects — intent and execution are fused together

**Remedy:** Read `notes/making-intent-explicit.md` and apply its guidance.

### D4. Decisions Buried in Execution
Look at code paths where complex decisions and their execution are interleaved — request handlers with branching logic, query builders that fire queries as they construct them, workflow orchestrators that act as they decide, multi-step transformations that mutate while choosing. Is the decision reified as an inspectable data structure — a plan, AST, command list, instruction sequence — before any side effect runs? Imagine adding a "preview," "dry-run," or "explain" mode. Does the existing structure support it, or would it require duplicating the whole flow?

**You have this problem if:**
- Complex branching and side effects are interleaved in one function
- There is no value a reader can inspect to see "what we decided to do" before it happens
- Testing a decision requires running side effects, or mocking them out
- The system cannot be dry-run, logged, or optimized because no plan exists as data
- Intermediate decisions live only as control flow, not as values with names and types
- Adding a "preview," "explain," or "optimize" mode would mean duplicating the flow rather than reading and transforming a plan

**Remedy:** Read `notes/making-intent-explicit.md` and apply its guidance.

### D5. Discarded Evidence
When data enters your system (user input, API response, file contents), is there a single point where it's parsed into a well-typed internal representation?

**You have this problem if:**
- You find null checks, type guards, or `if (!x)` scattered deep in interior logic
- Validation functions return `boolean` — downstream code has no *proof* of what was validated
- The same field is checked for validity in multiple places
- Functions deep in the core handle cases like "what if this field is missing?" that should be impossible by that point

**Remedy:** Read `notes/making-evidence-explicit.md` and apply its guidance.

### D6. Flat Domain / Missing Vocabulary
Look at the core types and modules of your system. Can you see distinct layers of abstraction? At the lowest layer, small precise primitives? At higher layers, richer compound concepts?

**You have this problem if:**
- Your domain is a flat collection of similarly-sized types and functions with no clear hierarchy
- You find yourself writing the same 3-4 line pattern in multiple places but there's no name for it
- High-level business functions are written in terms of low-level primitives — there's no intermediate vocabulary
- Reading a top-level function requires understanding all the implementation details beneath it

**Remedy:** Read `notes/growing-a-language.md` and apply its guidance.

### D7. Shallow Boundaries
Look at your module boundaries. For each module, ask: what design decision does this module hide? Does it hide a meaningful design decision behind an interface narrower than its implementation?

**You have this problem if:**
- Modules correspond to *process steps* ("first we do X, then Y") rather than *design decisions* they hide
- Some modules are so thin they're just pass-through wrappers — their interface is as complex as their implementation
- Some modules hide multiple unrelated decisions
- Changing one module's internals forces changes in its callers (leaky abstraction)

**Remedy:** Read `notes/making-boundaries-explicit.md` and apply its guidance.

### D8. Modules Organized Around Verbs, Not Types
Look at your module and file boundaries. For each significant module, ask: what type does this module own? Does each major module own a major data type — gathering its construction, validation, and transformation — or is it a bag of verbs anchored to no noun?

**You have this problem if:**
- Module names are verbs or roles: `userService`, `notificationManager`, `requestHandler`, `*Utils`, `*Helpers`
- A module exports many functions but defines no type — its vocabulary is entirely verbs
- A single domain type's lifecycle — construction, validation, transformation, queries — is scattered across multiple "service" or "manager" modules
- You can't point to the module that "owns" a domain concept; its type lives in one file but the functions that operate on it live elsewhere
- Adding a new operation on a type means choosing which existing service to bolt it onto, with no clear home
- Modules correspond to layers or roles ("controllers," "services," "repositories") rather than to the domain concepts they manipulate

**Remedy:** Read `notes/type-centric-modularization.md` and apply its guidance.

### D9. Information Loss
You have a concept (a user event, a domain operation, a data type) and you need to decide: should it stay as one thing or be broken into parts? Are concepts split where they cleave naturally, and bundled only when their parts truly belong together?

**You have this problem if (splitting too eagerly):**
- Downstream consumers keep needing context that was lost when the concept was decomposed
- Two pieces of data are always passed around together but live in different structures
- A split created two things that can't be understood independently — they only make sense as a pair
- A lower-level module has tangled control flow with many branches and heuristics — and when you trace back, it turns out the ambiguity was introduced by an upstream decomposition that stripped away constraining context. The module is pattern-matching its way back to information that was already available before the split

**You have this problem if (bundling too much):**
- Consumers only need one part of a compound concept but are forced to depend on the whole thing
- Different parts of a bundled concept change at different rates or for different reasons
- Adding a new consumer means they must understand the full bundle even though they only use a slice

**Remedy:** Read `notes/cohesion-and-decomposition.md` and apply its guidance.

### D10. Working Memory Overflow
Read through a function or module. Count the things you must hold in your head simultaneously: variables in scope, branches in flight, implicit state, conventions to remember.

**You have this problem if:**
- The count exceeds 4-5 simultaneous concerns
- You find yourself scrolling back up to remember what a variable held
- A function has 3+ levels of nesting (if inside if inside loop)
- A single loop body handles multiple unrelated transformations
- Understanding a line of code requires knowing something that happened in a distant part of the file

**Remedy:** Read `notes/the-readers-experience.md` and apply its guidance.

### D11. The Extract-Function Dead End
You have a large, unwieldy function or module. You try to improve it by extracting pieces into smaller functions. But afterward, nothing feels meaningfully better — you've segmented the code, not simplified it.

**You have this problem if:**
- Your refactoring consists entirely of extracting chunks into helper functions that are only called from one place
- The extracted functions take many parameters — they need nearly the full context of the original function to do their work
- You can't name the extracted functions well — they end up as `processStep1`, `handlePartA`, or `prepareData`
- After extraction, understanding the code still requires reading all the pieces together — the helpers can't be understood independently
- A file has many small functions, but no *types* — the module's vocabulary is entirely verbs (functions), with no nouns (data types) to anchor them

**Remedy:** Read `notes/type-centric-modularization.md` and apply its guidance.

### D12. Scattered Knowledge
Pick any convention in your codebase — a naming rule, a directory layout, a policy, a format assumption. Is it *defined* somewhere, or only *followed* in many places?

**You have this problem if:**
- A rule is encoded through repetition — many places follow it, no place defines it
- Changing the rule requires shotgun edits with no compiler help finding what you missed
- A new reader can only learn the convention by inferring from scattered instances

**Remedy:** Read `notes/making-knowledge-explicit.md` and apply its guidance.

### D13. Re-Litigated Invariants
Pick a property the code relies on as data flows through it — that an array is sorted, that a record has been enriched, that two fields are in a valid combination. Is that property *established once and carried in the type*, or *re-derived by every consumer*?

**You have this problem if:**
- A flag describing the data's shape (`'raw' | 'normalized'`, `'enriched' | 'unenriched'`) is threaded through multiple function signatures, and consumers branch on it
- Two or more downstream consumers ask the same question about the data and could fall out of sync
- Two fields must be in valid combinations and the relationship is enforced by reading code conventions, not by the type system
- The same property is re-sorted, re-checked, or re-derived at multiple call sites
- A bug arises because one consumer trusted an assumption that another consumer was responsible for upholding

**Remedy:** Read `notes/making-evidence-explicit.md` and apply its guidance.
