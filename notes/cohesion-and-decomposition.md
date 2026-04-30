# Cohesion and Decomposition: What Stays Together

When you decompose a concept, you may lose information that only existed in the relationship between the parts. The sign of this is tricky, difficult code that tries to make a decision based on multiple independent pieces of data — data that was once a single, coherent concept.

Consider: a user event triggers two downstream actions. You could immediately split the event into two action-specific messages and send them separately. But if those actions — or future actions — need to know about the *original event* (its timing, its user context, its intent), that information is lost. The split destroyed semantic content.

## Heuristics as a symptom of lost context

The most insidious form of this problem is that **the damage is invisible at the point where it hurts**. When you're working in a lower-level module that receives already-decomposed data, you don't know that relevant semantic information was stripped away upstream. All you see is that the behavior at your level is hard to pin down — there are too many possibilities, the control flow has too many branches, and the logic feels tangled.

To cope, you may find yourself writing heuristics — trying to *reconstruct* the original intention from the fragments you received. "If this field is present and that field has this pattern, then it probably came from scenario X." But the information was already there, concretely and explicitly, before it was decomposed. The heuristic is a symptom: it means somewhere upstream, a decomposition discarded context that this layer actually needed.

This is fundamentally hard to diagnose because the symptom (tangled code, heuristic-based decisions) appears far from the cause (premature decomposition at a distant point upstream). When you're working at the lower level, you can't see what you don't have. The explosion of possibilities is not intrinsic complexity — it's accidental complexity introduced by the decomposition that stripped away the constraining context.

**The diagnostic clue:** if a module's behavior is hard to specify, if it handles many cases through pattern-matching or heuristics, ask whether it's receiving the right data. Trace the data back upstream. Was there a richer, more concrete form of this data that was decomposed before it reached here? Could passing the original, unsplit concept have collapsed those many cases into a few obvious ones?

## Diagnostic questions for splitting vs. combining

1. **Do the parts have independent lifecycles?** If part A changes frequently but part B is stable, they want different modules. Split.
2. **Do consumers typically need both parts or just one?** If most consumers need only one part, the compound concept forces unnecessary coupling. Split.
3. **Does the relationship between the parts carry meaning?** If the parts are only meaningful *together* — if splitting them destroys information that consumers need — keep them together. The relationship is part of the domain.
4. **Can you split the structure but preserve the link?** Sometimes the answer is to split into independent concepts but maintain a reference (an ID, a parent type) that lets consumers reconstruct the relationship when needed.

## The inverse problem

A concept that bundles unrelated concerns forces every consumer to deal with the full bundle, even when they only need one part. This is the complecting that Hickey warns against ("Simple Made Easy," 2011): independent concerns woven into a single strand so they can no longer be reasoned about, tested, or changed independently. **Simple** means one role, one task, one concept, one dimension — not interleaved with anything else. Most legibility failures are places where independent concerns have been braided together; the remedy is structural un-braiding.

## Remedy

- **Default to keeping concepts together** at the point of origin. Let the data carry its full semantic content through the system.
- **Split at consumption points**, not at origin. When a specific consumer only needs part of a concept, let that consumer extract what it needs — don't pre-split at the source.
- **If you must split at origin**, ensure each piece carries enough context (or a reference back to the source) that downstream consumers can reconstruct the full picture.