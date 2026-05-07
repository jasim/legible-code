---
name: legible-code
description: >
  Diagnose and remedy code legibility problems by introducing domain types in place of
  primitives and booleans, pushing I/O to module edges, naming intermediate pipeline stages,
  reorganizing modules around data types instead of verbs, and consolidating duplicated logic
  into single authoritative definitions. Use when reviewing a PR, planning a refactor,
  designing a module/seam, or when the user signals that code is hard to reason about or asks where to split or merge a system.
license: MIT
metadata:
  author: Jasim A Basheer
  version: "2.1.0"
---

# Legible Code

## How to Operate

**Input:** One or more of: source files, a diff/patch, a plan or design document, a PR description, or a user brief. All of these are the **subject** — the artifact whose legibility you are diagnosing.

**Output:** A ranked list of the 2–5 highest-leverage legibility improvements. For each, state:

1. **Where** — exact location (file:lines, function name, or design section).
2. **Diagnostic** — the D-number and a one-line summary citing the specific issues that triggered the diagnostic. 
3. **Remedy** — drawn from the matching `notes/*.md` file. Read the note before recommending; do not paraphrase from memory.
4. **Leverage** — what becomes easier once fixed. If one fix dissolves several diagnostics, say so — those rank highest.

Do not produce a generic checklist. Do not rewrite the code unless asked. If nothing fires meaningfully, say so plainly.

### Procedure

Work in two passes. Do not start matching diagnostics on the first pass.

**Pass 1 — Skim.** Read the entire subject end-to-end before judging anything. Build a mental map of entry points and data flow (code), types and boundaries (code or design), decisions made vs. deferred (plans). Note suspicious regions — long functions, verb-named modules, scattered conventions, string-flag branching, in-place mutation, deep nesting, tangled I/O, unnamed repeated patterns. Produce a short list of **candidate hot spots**. Skip boilerplate, generated code, and unrelated parts.

**Pass 2 — Diagnose.** For each candidate, walk the diagnostics in order. A diagnostic fires only if you can point to specific evidence from the "You have this problem if" bullets — not on vibes. When several fire in the same region, prefer the lowest-numbered (upstream, often causes the others). Rank findings by leverage and emit output.

### Scope Rules

- **Diff or PR:** Weight findings toward issues the diff introduces or worsens. Don't pile on pre-existing problems unless they make the diff materially harder to review.
- **Plan or design:** Apply diagnostics to the proposed structure — which decisions are reified, where the seams are, what vocabulary is introduced, what knowledge would be scattered. Flag problems the plan would *bake in* if implemented as-written.
- **Files with no specific question:** Use the user's framing as the signal ("hard to test" → D3/D4/D5; "hard to change" → D7/D8/D13; "hard to follow" → D1/D2/D11).
- **Nothing fires:** "No high-leverage legibility issues found; here's what I checked" is a valid output. Do not invent problems.

---

## Diagnostics

Lower-numbered diagnostics are foundations; higher-numbered ones emerge from them. D13 (Scattered Knowledge) is cross-cutting. For the reasoning behind these diagnostics, see `INTRODUCTION.md`.

### D1. Narrative Incoherence
Look at your entry points — HTTP handlers, CLI commands, event subscribers, message consumers, job triggers. Any place where a user action or system event enters the codebase. Pick one entry point. Trace what happens from the moment it receives input to the final side-effect. Note how many files you pass through and whether each step is explicit or implicit.

**Examples:**
1. A user signup handler delegates to an auth service, which emits an event, caught by a listener in another module, which calls the email service — understanding one signup requires opening 7 files with no single place showing the outline of what happens.
2. A payment request flows through middleware, an event bus, a callback in a different file, and a state mutation in a service object — the path goes through implicit dispatch, not explicit calls, so a reader cannot trace trigger to outcome without reconstructing the event wiring.

**Remedy:** Read `notes/making-intent-explicit.md` and apply its guidance.

### D2. Invisible Data
As you trace the event path from D1, try to see the *data* at each stage. What shape does it have? What was added? What was transformed?

**Examples:**
1. A function modifies an object's fields and returns `void` — the caller must know to inspect the object afterward, but nothing in the type signature communicates that data was produced.
2. A pipeline where raw input is enriched, filtered, and transformed through deeply nested function calls with no named intermediate variables — the reader cannot see what shape the data has at any stage without tracing into each function.

**Remedy:** Read `notes/making-data-explicit.md` and apply its guidance.

### D3. Tangled Intent and Execution
Look at any function that makes a decision or performs a calculation. Does it also read from a database, call an API, or write to the filesystem? Can you exercise the decision logic without standing up the world around it?

**Examples:**
1. A function reads a user from the database, computes a discount based on business rules, and writes the discounted price back — you cannot test the discount logic without standing up a database or writing mocks, because the decision and the I/O are fused.
2. A request handler validates input, computes a pricing decision, and fires an API call — all interleaved. Extracting the pricing decision into a pure function (data in, decision out) with I/O pushed to the edges would make the logic testable without any mocks.

**Remedy:** Read `notes/making-intent-explicit.md` and apply its guidance.

### D4. Decisions Buried in Execution
Look at code paths where complex decisions and their execution are interleaved — request handlers with branching logic, query builders that fire queries as they construct them, workflow orchestrators that act as they decide, multi-step transformations that mutate while choosing. Is the decision reified as an inspectable data structure — a plan, AST, command list, instruction sequence — before any side effect runs? Imagine adding a "preview," "dry-run," or "explain" mode. Does the existing structure support it, or would it require duplicating the whole flow?

**Examples:**
1. A query builder constructs a SQL query and immediately fires it — there is no value the reader can inspect to see "what query was decided on" before execution. Adding a dry-run or explain mode requires duplicating the entire flow rather than reading a plan.
2. A workflow orchestrator decides which steps to run and runs them as it decides — branching and side effects are interleaved. Reifying the steps as a plan (a list, an AST) before execution would let you inspect, log, test, or optimize the plan independently of running it.

**Remedy:** Read `notes/making-intent-explicit.md` and apply its guidance.

### D5. Discarded Evidence
When data enters your system (user input, API response, file contents), is there a single point where it's parsed into a well-typed internal representation?

**Examples:**
1. A `validate()` function returns `boolean` — downstream code has no proof that the data was validated, so it adds its own null checks "just in case." The same field is guarded in three places, any of which could silently skip the check.
2. An HTTP handler receives raw JSON and passes it inward; null checks and `if (!x)` guards appear deep in business logic. A parser at the boundary converting raw input into a well-typed `ValidatedOrder` would make invalid states unrepresentable — eliminating all downstream null checks.

**Remedy:** Read `notes/making-evidence-explicit.md` and apply its guidance.

### D6. Flat Domain / Missing Vocabulary
Look at the core types and modules of your system. Can you see distinct layers of abstraction? At the lowest layer, small precise primitives? At higher layers, richer compound concepts?

**Examples:**
1. A payment module operates entirely on raw primitives — `number` for amounts, `string` for currencies, `string` for account IDs. There is no `Money`, `AccountId`, or `Fee` type. High-level functions are written in terms of low-level primitives with no intermediate vocabulary to compress repeated patterns.
2. The same 4-line pattern for "compute fee from amount and tier" appears in five places with no name. Extracting it as `Fee.compute(money, account)` creates a Layer 1 domain term built from Layer 0 primitives — reading a top-level function now reads like a sentence in the domain language, not like implementation details.

**Remedy:** Read `notes/growing-a-language.md` and apply its guidance.

### D7. Shallow Boundaries
Look at your module boundaries. For each module, ask: what design decision does this module hide? Does it hide a meaningful design decision behind an interface narrower than its implementation?

**Examples:**
1. A module named `RequestHandler` passes every parameter through to an underlying service — its interface is as complex as its implementation. It hides no design decision; it is just a pass-through wrapper that adds no depth.
2. Modules organized by process step — `Step1Validator`, `Step2Transformer`, `Step3Writer` — each hides lines of code but no meaningful design decision. Merging the validator and transformer behind a single `parse()` interface would create a deep module: simple surface, complex internals.

**Remedy:** Read `notes/making-boundaries-explicit.md` and apply its guidance.

### D8. Modules Organized Around Verbs, Not Types
Look at your module and file boundaries. For each significant module, ask: what type does this module own? Does each major module own a major data type — gathering its construction, validation, and transformation — or is it a bag of verbs anchored to no noun?

**Examples:**
1. A `userService` exports `createUser`, `validateUser`, `getUserName`, `updateUserEmail` — many verbs, no type. The `User` type lives in a shared types file while its lifecycle is scattered across `userService`, `authManager`, and `profileHelper`. Adding a new operation means choosing which service to bolt it onto, with no clear home.
2. A 150-line `processTransaction` function is split into `validateAmount()`, `computeFees()`, `buildLedgerEntries()` — single-call helpers that all share the same context and can't be understood independently. The type-centric approach identifies `Money`, `Account`, `Fee`, `LedgerEntry` as types, each gathering its own operations into a module; the function collapses to five lines.

**Remedy:** Read `notes/type-centric-modularization.md` and apply its guidance.

### D9. Information Loss (Premature Decomposition)
You have a concept — a domain event, a record, a value — that flows through the system. Look at where it is split into parts. Do the parts on their own carry less meaning than the whole, and do downstream consumers visibly try to put the meaning back together?

The signature is **reconstruction work in the consumer**: heuristics that approximate what the original record stated exactly, joins that re-fetch context the upstream split discarded, branch explosions where a single upstream variant becomes many downstream cases the consumer cannot distinguish.

**Examples:**
1. A `UserCreated` event is immediately split into `SendWelcomeEmail` and `CreateBillingAccount` messages — but the billing module later needs the original signup context (referral source, campaign) to determine the plan. It resorts to heuristics ("if the user signed up within 7 days, treat as referral") to reconstruct intent that was explicit before the split stripped it away.
2. A lower-level module has tangled control flow with many branches trying to determine which scenario produced the data — when you trace back, the ambiguity was introduced by an upstream decomposition that stripped away the constraining context. The module is pattern-matching its way back to information that already existed before the split.

**Remedy:** Read `notes/removing-meaning-causes-fragile-heuristics.md` and apply its guidance.

### D10. Invalid Bundling
You have a record, struct, or class that carries multiple fields. Look at when each field becomes valid, who reads it, and how often its shape changes. Are all the fields actually one concept — co-created, co-read, co-evolving — or are they distinct concepts that have been glued together because they happened to travel near each other?

The signature has two faces, and they are the same problem seen from two angles:

**(a) The type lies at construction.** A field is typed as present but is populated only mid-construction, forcing `undefined as unknown as T` casts, `!` non-null assertions, or a "patch in later" pattern behind a ref or setter.

**(b) The type admits invalid combinations.** Take the fields whose presence/validity is genuinely independent in the type and enumerate their combinations. If the type permits `2^n` shapes but only a handful are domain-valid, the type is failing the *make illegal states unrepresentable* principle. Optional fields that are "always set together" (or "never set unless this other one is") are the giveaway: the type is stating an independence the domain denies.

Other supporting signals:
- Consumers reach into only one slice of the record and ignore the rest; different slices are read by entirely different modules.
- A schema change to one slice forces recompilation, redeployment, or test churn in consumers that never touched that slice.
- Code defends against impossible combinations with runtime guards (`if (this.conn && !this.child) throw ...`) — a re-litigation of an invariant the type should have prevented.

**Examples:**
1. A `LiveSession` bundles session-domain state (`id`, `cwd`, `terminals`, lifetime = whole session) with the spawned agent process (`conn`, `child`, `stderrTail`, lifetime = only after `createAgentConnection` returns). If the three process fields are typed as non-optional, construction has to lie (`undefined as unknown as T`); if they are typed as optional, the type admits eight combinations of present/absent across the three, but the domain only ever wants two — *all three present* (after spawn) or *all three absent* (before spawn). The remaining six are representable but invalid, and downstream code has to either guard against them or assume they don't happen. Splitting into `SessionState` and `AgentConnection`, bundled as `LiveSession` only at the manager seam (sibling of the DB-row and wire-shape representations of a session), collapses the eight-state space down to the two legal ones — the invalid combinations stop being expressible at all.
2. A `Document` type bundles metadata, content, and rendering config. Every consumer depends on the whole even when it only needs one slice; when rendering config changes schema, the search indexer breaks even though it never reads rendering config. The slices have different audiences and different change rates — they are not one concept.
3. A `User` record carries profile data, per-request auth state, and eventually-consistent analytics counters. Three different lifetimes pretending to be one. Any code that updates `User` has to reason about all three; analytics writes contend with profile reads.

**Remedy:** Read `notes/bundling-distinct-concepts-causes-impossible-types.md` and apply its guidance.

### D11. Working Memory Overflow
Read through a function or module. Count the things you must hold in your head simultaneously: variables in scope, branches in flight, implicit state, conventions to remember.

**Examples:**
1. A function with 3 levels of nesting — an `if` inside a `try` inside a `for` loop — where the loop body handles validation, transformation, and error logging simultaneously. The reader must track which branch they're in, which data is being transformed, and what errors are being caught, all at once.
2. A state object is mutated by a distant helper function, then read by another function later — understanding the second function requires knowing what the first one did. Action at a distance forces the reader to hold both locations in working memory simultaneously.

**Remedy:** Read `notes/the-readers-experience.md` and apply its guidance.

### D12. The Extract-Function Dead End
You have a large, unwieldy function or module. You try to improve it by extracting pieces into smaller functions. But afterward, nothing feels meaningfully better — you've segmented the code, not simplified it.

**Examples:**
1. A 200-line function is refactored into `step1()`, `step2()`, `step3()` — each called once, each taking 8 parameters, none independently understandable. The code is segmented but not simplified; understanding it still requires reading all the pieces together.
2. After extracting `validateAmount()`, `computeFees()`, and `buildLedgerEntries()` from a transaction processor, nothing improves — the helpers share the same context and can't compose or reuse. The real solution is identifying domain types (`Money`, `Fee`, `LedgerEntry`) and colocating each type with its operations, so the function collapses into a short composition of domain terms.

**Remedy:** Read `notes/type-centric-modularization.md` and apply its guidance.

### D13. Scattered Knowledge

Often called code duplication; but more accurately - knowledge duplication. It includes knowledge about how specific operations, data transformations, policies and parsing/validation is done. It also includes conventions - naming rules, format assumptions, ordering etc. 

Are they *defined* somewhere - in one authoritative source, or duplicated/followed in many places with? If the knowledge is scattered then it is hard to have a coherent understanding and we lose confidence in our ability to work with the codebase.  

**Examples:**

1. Every API response handler repeats the same field-mapping logic — `data.first_name` → `firstName` — but no single function defines the mapping. Changing the API's field naming convention requires a shotgun edit with no compiler help finding what you missed.
2. A directory layout convention (controllers in `src/controllers/`, services in `src/services/`) is followed by 30 modules but defined nowhere. A new developer can only learn it by inferring from scattered instances, and a violation goes undetected.

**Remedy:** Read `notes/making-knowledge-explicit.md` and apply its guidance.

### D14. Re-Litigated Invariants
Pick a property the code relies on as data flows through it — that an array is sorted, that a record has been enriched, that two fields are in a valid combination. Is that property *established once and carried in the type*, or *re-derived by every consumer*?

**Examples:**
1. An array is sorted in one function, but every downstream consumer re-sorts it "just to be safe" — three call sites call `.sort()`, and if one is missed, a subtle bug appears. A `ChronologicallySorted<T>` type, constructed once by a factory, would make the sortedness guarantee structurally undeniable and eliminate all re-sorting.
2. A record has `status: 'active' | 'expired'` and `expiresAt: Date | null` — two consumers independently check `if (status === 'active') { use(expiresAt!) }`. If someone adds a new status, both checks must be updated in lockstep. A discriminated union `ActivePlan | ExpiredPlan` makes the valid combination structurally undeniable, removing the question from downstream code entirely.

**Remedy:** Read `notes/making-evidence-explicit.md` and apply its guidance.
