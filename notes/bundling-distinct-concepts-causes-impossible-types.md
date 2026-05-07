# Bundling distinct concepts forces the type to admit invalid states

A type defines a *representable set*: the set of values the compiler will let you construct. The domain defines a *valid set*: the values that make sense in the problem. The "make illegal states unrepresentable" rule says these two sets should match — every representable value is valid, every invalid value is unrepresentable.

When you bundle multiple distinct concepts into one record, the representable set inflates. Each independent field multiplies the combinations the type permits, but only a small fraction of those combinations are domain-valid. The gap between the two sets is where the bugs live, and where every consumer is forced to re-litigate invariants the type should have carried.

This is the **dual** of premature decomposition. There, you split a single concept too early and consumers reconstruct it with heuristics. Here, you bundle distinct concepts too tightly and the type ends up describing a shape that doesn't honestly exist at any single moment — or worse, a shape that admits combinations the domain forbids. The shared underlying choice is **how to factor the concept**; both diagnostics fire when that choice is wrong.

## Two faces of the same defect

The bug presents as two symptoms that turn out to be the same thing:

**(1) Construction-time dishonesty.** You write `undefined as unknown as T`, or `!`, or you defer the whole record behind a ref that's still `null` when the first reader fires. The type says the field is `T`, but at this moment in the lifetime there is no `T` to assign. The cast is not a TypeScript quirk — it is the type system telling you that the bundle mixes lifetimes and the compiler cannot honestly type the in-progress state.

**(2) Cartesian-product blowup.** You mark the staged fields as `Optional<T>` instead of casting. Now construction is honest, but each independent optional field doubles the representable set. Three optional fields admits eight combinations of present/absent, but the domain probably wants two: *all present* (the fully-spawned state) or *all absent* (the pre-spawn state). The other six are representable nonsense.

These are not two problems. They are two outcomes of the same wrong factoring: you tried to make one type carry concepts with different lifetimes, and the type system gave you a choice between *lying about the present state* (1) or *admitting impossible combinations* (2). Both push invariants out of the type and into the consumer.

## Why mixed lifetimes break types

Types describe invariants that hold *for the lifetime of a value*. A `User` of type `User` should be a `User` from construction to garbage collection — every method that reads it must be able to trust every field. When you bundle a field that becomes valid only later in the record's life, you have already broken that contract; the casts and the optionality are just two ways of paying the bill.

The honest representation has two types, each with a coherent lifetime. The pre-spawn concept is fully populated from the moment its type exists; the post-spawn concept is fully populated from the moment *its* type exists. If the two genuinely co-travel through some seam of the system, you bundle them *there*, with a name that describes the co-traveling unit — but you do not pretend they are one concept at origin. At the seam, the bundle's representable set matches its valid set: both parts are present, by construction.

## Diagnostic tests for invalid bundling

1. **The representable-vs-valid test.** Enumerate the independent present/absent (or otherwise-orthogonal) field positions. Count the combinations the type permits. Count the combinations the domain actually allows. If the first is materially larger than the second, the type is admitting illegal states.

2. **The honest-construction test.** Can you build a value of this type with every field populated by a real value computed from real inputs, in a single expression, with no casts, no placeholders, no "patch in later"? If not, the bundle is mixing concepts with different lifetimes.

3. **The optional-but-correlated test.** Do you have several `Optional<T>` fields that are, in practice, always set together and always unset together? Each pair like that is one bit of real domain state expressed as two bits of representable state — the spare bit is exactly the gap between representable and valid.

4. **The runtime-guard test.** Are there `if (this.a && !this.b) throw` checks, or invariant assertions, defending against combinations the type permits but the domain forbids? Each such guard is a re-litigation of an invariant the type should have carried.

5. **The slice-access test.** For each consumer of the type, list which fields it reads. If consumers cluster cleanly into disjoint slices — the auth code only ever touches the auth fields, the billing code only ever touches the billing fields — those slices are separate concepts. The bundle is a coincidence of storage, not a domain fact.

6. **The change-rate test.** When one slice's schema changes, do consumers of unrelated slices have to be rebuilt, redeployed, or re-tested? That ripple is accidental coupling — the change rate is per-slice but the type is whole-record.

7. **The seam test.** Even if two concepts must travel together through *some* part of the system, ask whether they travel together *everywhere*. If they do — same lifetime, same readers, same change drivers — keep them bundled. If they only co-travel in one zone (e.g., a manager that owns both), bundle only there: define the bundle at the seam, and let the surrounding code see the two parts separately.

# Remedy

- **Shrink the representable set to the valid set.** This is the governing rule; the moves below are how you do it. After every refactor, re-run the representable-vs-valid test: the gap should be smaller.

- **Factor at lifetime boundaries.** When two slices have different moments at which they become valid, they are different types. Give each its own type, owned by the module responsible for its lifecycle. Each of the new types is fully populated from construction.

- **Use a discriminated union when the bundle has distinct phases.** If a value moves through two or three named states (e.g., `Pending | Running | Closed`), encode the phase as a tag and let each variant carry exactly the fields valid in that phase. This is the textbook *make illegal states unrepresentable* form: the representable set is the disjoint union of per-phase valid sets, with no Cartesian-product slack.

- **Bundle at the seam, not at origin.** When two cleanly-typed parts do genuinely co-travel through one zone of the system, define a named bundle (e.g., `LiveSession { state: SessionState; conn: AgentConnection }`) at exactly that seam. The bundle exists where it earns its keep — not as the universal shape of the data.

- **Refuse the cast.** When you find yourself writing `undefined as unknown as T`, `!`, or "I'll patch this in after construction", treat it as a structural signal rather than a syntax problem. The remedy is almost never to find a cleverer cast; it is to split the type so that construction is honest.

- **Make optionality meaningful.** If a field is `Optional<T>`, the optionality should reflect a real domain question (the user *may* have a phone number). Optionality used to paper over construction order is a smell; convert it into two types, a discriminated union, or a presence flag at the seam.
