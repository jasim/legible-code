# Making Knowledge Explicit

When knowledge — a convention, policy, structural assumption, domain rule — exists only as a pattern repeated across files, nobody can confidently understand, change, or verify it. There could be exceptions in places, and nobody will notice when localized inconsistencies are introduced.

A single authoritative definition fixes this. Consumers depend on the definition through its interface instead of reimplementing the knowledge themselves. You can confidently change behaviour in one place have it work robustly everywhere.

## DRY is about knowledge, not characters

"Every piece of knowledge must have a single, unambiguous, authoritative representation within a system." (Hunt & Thomas, *The Pragmatic Programmer*.) This is often misread as "don't write similar-looking code" — but the point is about *knowledge*, not *characters*. Two pieces of code that happen to look alike but encode different facts should not be merged; two pieces of code that encode the same fact in different places must be consolidated.

## The confidence problem

The main cost of scattered knowledge is loss of confidence. When a convention is followed in twenty places but defined in zero, a reader can never be certain they have the full picture. The lurking question: *what if there's an exception I haven't seen?*

This compounds. Developers are reluctant to change what they can't fully map. The system resists change because nobody knows the blast radius.

One authoritative definition restores confidence. The developer changes it in one place; the compiler (or tests, or the single call site) shows what else must adapt.

## Remedy

- **Name the knowledge.** When a convention is followed across multiple places, ask: is it *defined* anywhere? If not, it needs a home.
- **Give it a single definition:**
  - Structural convention → schema or builder function
  - Naming rule → generator function
  - Policy → policy function consulted, not reimplemented
  - Configuration assumption → constant or config object
  - Domain constraint → type whose construction enforces it (see [Making Evidence Explicit](./making-evidence-explicit.md))
- **Make consumers depend on the definition.** If you define a directory layout schema but every module still hardcodes its own paths, you've created documentation, not consolidation.
- **Test the definition, not each instance.** One definition means one focused test. Scattered knowledge requires a test per instance — and you'll miss the ones you don't know about.
