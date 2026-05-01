# Making Evidence Explicit: Types as Guarantees

Use the type system to convert information about invariants in a piece of data into a type guarantee that is statically undeniable. This way the invariant check need only be done once, and downstream code need not re-verify them ever again. This principle operates everywhere: at the boundaries of the system, deep inside domain code, and at every seam where data changes shape or meaning.

## Use parser functions instead of validations

A validation function is one that returns a boolean depending on whether the check failed or succeeded. This however carries no structural guarantee which can be relied on by code that is not in its immediate vicinity. This is called **boolean blindness**. 

It is an instance of a broader pattern: **re-litigation**. Whenever an invariant is not encoded in the type system, every consumer that depends on it must independently know about it, uphold it, and re-establish it. The pattern recurs at every level of the system:

- At the boundary, unvalidated raw data forces interior code to check for null or invalid values.
- In the domain, an unlifted invariant forces each consumer to independently uphold it.
- At runtime, a guard that callers must remember to invoke leaves adherence to convention — and that adherence decays as the codebase grows.

In every case the remedy is the same: make the evidence explicit so the type system carries the proof, eliminating re-litigation.

### Boundary invariants

These invariants hold from the moment data enters the system. The boundary accepts data from the external world — network calls, user input, file I/O, database queries — and none of it carries type-level guarantees until the program establishes them. The invariant here is that the data conforms to a valid internal shape, and this is the "Parse, Don't Validate" pattern: parsing *converts* raw input into a richer type where invalid states are unrepresentable, failing explicitly if the input is malformed.

The boundary protocol has three steps:

1. **Define an internal type** that represents the *valid* shape of the data. Invalid states should be unrepresentable in this type.
2. **Write a parser** at the boundary that converts raw input into this type, failing explicitly if the input is malformed.
3. **Trust the type downstream.** Once data has been parsed, no further validation should be necessary.

### Interior invariants

Not all invariants are present at the boundary. Some emerge only inside domain code as data is enriched, sorted, joined, or filtered. These come in two sub-kinds:

- **Shape invariants** — the array is sorted, the list is non-empty, every record has been enriched with a `tenant_id`.
- **Relational invariants** — two or more values are constrained to be in valid combinations: `status` and `expires_at`, `start` and `end`, currency and amount.

Like boundary invariants, interior invariants can be lifted into types — the difference is simply that the evidence becomes available later, inside the domain rather than at the edge.

### Exterior invariants

Some invariants require evidence that lives outside the program's static world entirely. No type within the program can prove them at compile time because the truth depends on live data or external systems, and can even change between calls.

Exterior invariants cannot use the strongest enforcement tier, because the evidence simply isn't available at construction time.

## The Enforcement Spectrum

Invariants can be enforced with varying degrees of compiler support. Try to always use the strongest tier that is expressible — stepping down only when a higher tier cannot capture the evidence.

### Tier 1: Nominal type at construction

The strongest form of enforcement is a type whose existence *is* the invariant. The type's only constructor establishes the invariant. Because the compiler enforces the guarantee at every downstream call site for free, re-litigation is impossible.

This tier applies to boundary invariants — the parser produces the nominal type — and to interior shape and relational invariants when the evidence is available at construction.

**For shape invariants,** use a discriminated record with a `kind` tag (e.g. `{ kind: 'chrono', rows: T[] }`) or a class with a private constructor and a `make` / `normalize` factory.

**For relational invariants (co-varying fields),** replace independent fields with a single compound type whose shape can only express valid combinations.

Example: a discriminated union over `status` so that `expires_at` exists only when `status === 'active'`:

```ts
type Subscription =
     | { status: 'active'; expires_at: Date }
     | { status: 'cancelled' }
```

Example: a `DateRange` whose smart constructor refuses `start > end`:

```ts
type DateRange = { start: Date; end: Date }
export const makeDateRange = (start: Date, end: Date): DateRange.t | Error =>
       start > end ? Error('start > end') : { start, end }
```

After this invariant lifting, downstream code that previously branched on a flag can expect a typed argument instead. The branching is no longer necessary because the type already answers the question.

### Tier 2: Assertion function that narrows the type

When the invariant depends on runtime data — as with exterior invariants — Tier 1 is not possible because the evidence isn't available at construction. The next best option is an assertion function that checks the invariant at runtime and, on success, narrows the type so the compiler knows the value is now valid:

```ts
type ExistingCustomerId = CustomerId & { readonly __existing: unique symbol };

function assertCustomerExists(
  id: CustomerId,
  tenant: TenantId,
  registry: CustomerRegistry,
): asserts id is ExistingCustomerId {
  if (!registry.has(id, tenant)) throw new CustomerNotFound(id);
}

// Call site:
assertCustomerExists(id, tenant, registry);
chargeCustomer(id); // chargeCustomer requires ExistingCustomerId
```

After the assertion returns, the compiler tracks `id` as `ExistingCustomerId`. Any downstream function that demands the branded type accepts it without re-checking — the evidence has been promoted into the type. The same shape works with a `function isX(v): v is X` predicate when the unhappy path is a branch rather than a throw.

### Tier 3: Pure throwing guard

Sometimes validity cannot be captured in a type at all — there is no meaningful brand, no richer type to return, only a runtime fact that must be true at this moment. In that case, fall back to a plain throwing guard whose only contract is success-or-throw:

```ts
function assertWithinRateLimit(key: RateLimitKey, bucket: TokenBucket): void
```

This form has no type effect. It is pure convention: every call site that needs the invariant calls the guard; nothing in the type system enforces that they do. Re-litigation is unconstrained — adherence decays as the codebase grows. To keep the convention durable, give the guard a single canonical home, a name that states the invariant, and a docstring that lists the call sites that depend on it. Treat a missing call as a review-time defect.

## Where to Lift

Lift the invariant at the *earliest stage where it can be made true*.

- For sortedness or canonical form, that means at the parse boundary, before any consumer sees the data.
- For enrichment, it means in the function that performs the enrichment, returning the enriched type.
- For relational constraints between fields, it means in the smart constructor of the compound type, called wherever the related fields first become known together.
- For exterior invariants, it means at the call site immediately before the code that depends on the invariant, via an assertion function.

## The Deletability Test

After lifting, audit the downstream code. Every conditional that branched on the question, every defensive re-check, every comment explaining "we know this is sorted because…" should be deletable. If they aren't, the invariant has not actually been lifted — the type is decorative, and the question is still being re-litigated.

This test applies uniformly across the system. If you find yourself checking for null or invalid values deep in the interior, the boundary parser is incomplete. If interior code re-checks an invariant that a nominal type already guarantees, the type isn't carrying its weight. In each case, the surviving check tells you exactly where the evidence is still missing.

## Two Universal Rules

Regardless of which tier you use, two rules apply across the entire enforcement spectrum:

1. When an invariant doesn't hold, that is an exceptional situation — so treat it as one. Return an appropriate type or throw; never return a boolean that the caller can ignore. The goal is a system free of ambiguities, where code can make decisions with confidence because every path either carries proof or stops.

2. **One canonical definition per invariant.** Whether the invariant is established by a nominal type's constructor, a narrowing assertion function, or a plain throwing guard, there should be one authoritative place where it is defined.
