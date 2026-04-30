# Making Evidence Explicit: Types as Guarantees

Validation *checks* that data is well-formed but throws away the evidence. A function that returns `boolean` tells the caller "yes" or "no" but doesn't encode *what was verified* into the type system — this is **boolean blindness** (Harper, 2011). Downstream code is left to trust on faith — or to re-check.

The remedy is to make the evidence explicit: convert data into types where the guarantee is structurally undeniable. This principle operates at two boundaries — the system edge and inside the pipeline.

## At the system boundary: Parse, Don't Validate

Parsing *converts* data into a richer type where invalid states are unrepresentable (Minsky, "Make Illegal States Unrepresentable"; King, "Parse, Don't Validate"). After parsing, downstream code can't encounter surprises — the type system guarantees what was verified.

The boundary is where the messy world meets your clean domain. Everything inside the boundary speaks the language of your domain types.

For every system boundary (HTTP request handler, file reader, database query result, user event):

1. **Define an internal type** that represents the *valid* shape of the data. Invalid states should be unrepresentable in this type.
2. **Write a parser** at the boundary that converts raw input into this type, failing explicitly if the input is malformed.
3. **Trust the type downstream.** Once data has been parsed, no further validation should be necessary. If you find yourself checking for null or invalid values deep in the interior, the boundary parser is incomplete.

## Inside the pipeline: Lifted Invariants

Some invariants don't live at the system boundary; they emerge *inside* the pipeline as data is enriched, sorted, joined, or filtered. Two kinds matter:

- **Shape invariants** — the array is sorted, the list is non-empty, every record has been enriched with a `tenant_id`.
- **Relational invariants** — two or more values are constrained to be in valid combinations: `status` and `expires_at`, `start` and `end`, currency and amount.

When such an invariant is not lifted into the type system, downstream consumers must each independently know about and uphold it. The cost is paid twice: once in the bugs, and once in the cognitive load of every reader who must reconstruct what is true about the data at this point in the program.

### Concrete nominal type (preferred)

Define a type whose existence *is* the invariant. The type's only constructor establishes the invariant; downstream code receives the type and may not re-decide.

For shape invariants: a discriminated record with a `kind` tag (e.g. `{ kind: 'chrono', rows: T[] }`) or a class with a private constructor and a `make` / `normalize` factory.

For co-varying fields: replace the independent fields with a single compound type whose shape can only express valid combinations. A discriminated union over `status` so that `expires_at` exists exactly when `status === 'active'`. A `DateRange` whose smart constructor refuses `start > end`. An `Amount` that pairs value and currency so currency-less arithmetic is unrepresentable.

After lifting, downstream code that previously branched on a flag now demands the typed argument. Any function that does not preserve the invariant cannot return the type, so the place where order or shape might be lost is exactly the place a reviewer looks.

### Reusable guard (fallback)

When the constraint spans values held in different domains and cannot be co-located in a single type — a foreign-key relationship between records owned by different services, a constraint that requires runtime context to evaluate — write *one* guard function that asserts the invariant and **throws loudly** on violation. Call this guard at every boundary where the invariant must hold.

Avoid silent boolean returns: a `boolean` discards the evidence and lets callers ignore it. The guard either succeeds or throws.

This is a fallback, not a peer. A nominal type is checked at compile time at every call site for free; a guard is checked only at sites that remember to call it. Reach for the guard when the type approach is genuinely infeasible, not when it would be slightly more work.

### Where to lift

Lift the invariant at the *earliest stage where it can be made true*:

- For sortedness or canonical form: at the parse boundary, before any consumer sees the data.
- For enrichment: in the function that performs the enrichment, returning the enriched type.
- For relational constraints between fields: in the smart constructor of the compound type, called wherever the related fields first become known together.

After lifting, audit downstream code: every conditional that branched on the question, every defensive re-check, every comment explaining "we know this is sorted because…" should be deletable. If they aren't, the invariant has not actually been lifted — the type is decorative, and the question is still being re-litigated.