# Making Evidence Explicit: Types as Guarantees

A validation function *checks* that data is well-formed and returns a boolean, but the information is not encoded into the type system. This is called **boolean blindness**. Downstream code will have to trust that the control flow that brought it the data would've done the verification already, or if it has to be sure, it will have to re-validate.

The remedy is to make the evidence explicit: convert data into types where the guarantee is structurally undeniable. This principle operates everywhere - in the boundaries of the system as well as inside domain code.

## At the system boundary: Parse, Don't Validate

Parsing *converts* data into a richer type where invalid states are unrepresentable (Minsky, "Make Illegal States Unrepresentable"; King, "Parse, Don't Validate"). After parsing, downstream code can't encounter surprises — the type system guarantees what was verified.

The boundary accepts data from the external world - this could be network calls, user input, file I/O, or database queries. For every one of them, we have to:

1. **Define an internal type** that represents the *valid* shape of the data. Invalid states should be unrepresentable in this type.
2. **Write a parser** at the boundary that converts raw input into this type, failing explicitly if the input is malformed.
3. **Trust the type downstream.** Once data has been parsed, no further validation should be necessary. If you find yourself checking for null or invalid values deep in the interior, the boundary parser is incomplete.

## Inside the domain: Lifted Invariants

Some invariants don't live at the system boundary; they emerge *inside* domain code as data is enriched, sorted, joined, or filtered. Two kinds matter:

- **Shape invariants** —  eg: the array is sorted, the list is non-empty, every record has been enriched with a `tenant_id`.
- **Relational invariants** —  two or more values are constrained to be in valid combinations: `status` and `expires_at`, `start` and `end`, currency and amount.

When such an invariant is not lifted into the type system, downstream consumers must each independently know about and uphold it.

### Remedies

#### Concrete nominal type

Define a type whose existence *is* the invariant. The type's only constructor establishes the invariant; downstream code receives the type and may not re-decide.

For shape invariants: a discriminated record with a `kind` tag (e.g. `{ kind: 'chrono', rows: T[] }`) or a class with a private constructor and a `make` / `normalize` factory.

For co-varying fields: replace the independent fields with a single compound type whose shape can only express valid combinations. 

Example: a discriminated union over `status` so that `expires_at` exists only when `status === 'active'`: 

```ts 
type Subscription =
     | { status: 'active'; expires_at: Date }
     | { status: 'cancelled' }
```

Another example: a `DateRange` whose smart constructor refuses `start > end`:

```ts
type DateRange = { start: Date; end: Date }
export const makeDateRange = (start: Date, end: Date): DateRange.t | Error =>
       start > end ? Error('start > end') : { start, end }
```

After this invariant lifting is done, downstream code that previously branched on a flag can expect a typed argument instead. Any function that does not preserve the invariant cannot return the type, thus guaranteeing the invariants. 

### Reusable runtime guard as a fallback

Some invariants can't be lifted into a type because the evidence lives outside the program's static world. Three common cases:

- **Cross-aggregate references.** An `Order` carries a `customerId`, but the `Customer` row lives in another table — or another service. The type system can't prove the foreign key resolves; only a lookup against live data can.

- **Runtime authorization.** "This caller may read this document" depends on the request's auth context and the document's ACL. Neither is known at compile time, and neither belongs in the document type.

- **External state.** "This S3 key exists", "this feature flag is on for this tenant", "this idempotency key has not been used" — the truth lives in another system and can change between calls.

For these, write *one* guard that asserts the invariant and throws on violation, then call it at every site where the invariant must hold:

```ts
// Throws CustomerNotFound if the FK does not resolve in the current tenant.
async function assertCustomerExists(id: CustomerId, tenant: TenantId): Promise<void>
```

Two rules:

1. **Throw, don't return a boolean.** If an invariant doesn't hold, it is an exceptional situation. We want our system to avoid ambiguities, so our code can make decisions with confidence.  
2. **One canonical guard per invariant.** When the rule changes there should only be place to change it. It also becomes a clear convention so that if a site forgets to call it, the omission is visible immediately.

Runtime guards are a fallback. If we were able to use the type system, then the compiler will check every call site for free. But a runtime guard is checked only at sites that remember to call it, and it is a convention, that has to be followed. Its adherence decays as the codebase grows. So reach for runtime guards only when the evidence genuinely cannot be made structural.

### Where to lift

Lift the invariant at the *earliest stage where it can be made true*:

- For sortedness or canonical form: at the parse boundary, before any consumer sees the data.
- For enrichment: in the function that performs the enrichment, returning the enriched type.
- For relational constraints between fields: in the smart constructor of the compound type, called wherever the related fields first become known together.

After lifting, audit downstream code: every conditional that branched on the question, every defensive re-check, every comment explaining "we know this is sorted because…" should be deletable. If they aren't, the invariant has not actually been lifted — the type is decorative, and the question is still being re-litigated.
