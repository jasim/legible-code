# Type-Centric Modularization

When code resists modularization — when "extract function" is the only tool you reach for and it yields no real improvement — the problem is almost never a lack of functions. It's a lack of *types*.

## Why "extract function" alone fails

Extracting functions without introducing types is *segmentation*, not *modularization*. You're cutting a long piece of code into labeled chunks, but:

- **No new abstraction is gained.** The chunks share all the same data — they don't reduce what a reader needs to know.
- **The functions can't compose or reuse.** They exist only to serve the original flow. No other caller would ever invoke them.
- **The functions don't hide a design decision.** They hide *lines of code*, not *complexity*. The interface (the parameter list) is as complex as the implementation.
- **Parameter lists explode.** Because no intermediate types bundle related data, each extracted function needs many arguments — often the same ones as its siblings.

The missing step is identifying the *things* the code is about — the domain concepts that deserve to be types — and letting those types pull related operations into modules.

## The type as organizing principle

In OCaml, every significant domain concept gets its own module. The module file contains a type — conventionally named `t` — along with every function that creates, transforms, validates, or inspects values of that type. The type *is* the organizing principle: the module boundary follows from the type, not from a process step or a feature grouping. You don't ask "what does this module *do*?" — you ask "what *thing* does this module define?"

This principle translates directly to TypeScript and other languages with structural type systems. The insight is structural: **a type and its operations form a natural module**. When you find yourself unable to decompose a large piece of code, it's usually because the intermediate domain types haven't been identified and named. Once you name them, the module boundaries reveal themselves — each type pulls its related operations into a cohesive unit.

A type is not just a data shape. It's a *concept in the domain* that has its own rules, its own invariants, its own operations. When you give it a module, you're saying: "this concept is important enough to have its own vocabulary." Everything about that concept — how to create it, validate it, transform it, query it — lives in one place.

This is what gives types their modularizing power. Functions alone don't tell you *what goes together* — any function can call any other function. But a type creates a gravitational center: operations that primarily concern this type belong here; operations that primarily concern a different type belong there. The type provides a criterion for cohesion that process-step decomposition lacks.

## Technique

1. **Find the types.** Read through the large block of code and ask: what are the distinct *things* being manipulated here? Look for clusters of fields that travel together, for data that undergoes its own validation or transformation, for concepts that the code treats as a unit even if no type definition exists yet.

2. **Create a module for each type.** Each type gets its own file containing:
   - The type definition itself
   - Creation functions (constructors, parsers, factories)
   - Transformations (functions that take a value of this type and return a new value)
   - Queries (functions that inspect or extract information)
   - Validation (if the type has invariants)

3. **Handle compound operations.** When a function operates on multiple types:
   - If one type clearly *drives* the operation, place the function in that type's module.
   - If no type dominates, or if the operation represents a distinct domain concept, create a separate module for it.

4. **Let the original code become a composition.** After extracting types and their operations, the original long function should collapse into a short sequence of calls into well-typed modules. The function's body now reads as a sentence composed of domain terms — which is exactly the layered vocabulary of [Growing a Language](./growing-a-language.md).

## Example

Before — a 150-line function that processes a financial transaction:

```typescript
function processTransaction(raw: any) {
  // 30 lines: validate and normalize the amount, currency, exchange rates
  // 20 lines: look up account, check balance, determine account type
  // 25 lines: compute fees based on transaction type, account tier, amount thresholds
  // 20 lines: build the ledger entries (debits, credits, fee entries)
  // 15 lines: format the result for the API response
}
```

Extracting functions yields `validateAmount()`, `lookupAccount()`, `computeFees()`, `buildLedgerEntries()`, `formatResponse()` — five functions, each called once, each needing most of the same context. Nothing is gained.

The type-centric approach asks: *what are the things?* Answer: `Money` (amount + currency + exchange context), `Account` (balance + tier + type), `Fee` (rule + computed amount), `LedgerEntry` (debit/credit + account + amount). Each becomes a module with its own creation, validation, and transformation functions. The original function becomes:

```typescript
function processTransaction(raw: RawTransaction) {
  const money = Money.parse(raw.amount, raw.currency)
  const account = Account.lookup(raw.accountId)
  const fees = Fee.compute(money, account)
  const entries = LedgerEntry.fromTransaction(money, account, fees)
  return TransactionResult.format(entries)
}
```

Five lines, each a single domain operation. The complexity didn't disappear — it moved into modules organized around types, where it can be understood, tested, and modified independently.

## Remedy

- When facing intractable code, **look for types first, not functions**. The types you identify become your module boundaries.
- **Colocate operations with their type.** Creation, validation, transformation, and queries on a type all belong in that type's module.
- **For compound operations**, determine the dominant type or create a dedicated compound module.
- **Avoid "helper function" files.** If a function doesn't have a clear type it belongs to, that's often a sign of a missing type — not a missing utils file.
- **Don't force opacity.** In structurally typed languages like TypeScript, you don't need OCaml's opaque types to benefit. The value is in the *organizational* principle — colocating a type with its operations — not in access control.