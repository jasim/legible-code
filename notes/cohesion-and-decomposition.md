# Cohesion and Decomposition: What Stays Together

When you decompose a concept, you may lose information that only existed in the relationship between the parts. The sign of this is tricky, difficult code that tries to make a decision based on multiple independent pieces of data — data that was once a single, coherent concept.

The precise operation is **projection**: mapping a richer type onto a poorer one. A projection `p: A → B` loses information when no inverse `p⁻¹: B → A` exists — when you cannot reconstruct the original from the projected form. This is the algebraic heart of the problem. Downstream code that tries to reconstruct `A` from `B` is trying to invert an uninvertible function. It will use heuristics, pattern-matching, and auxiliary queries — all approximations of information that was exact before the projection destroyed it.

Consider a user event that triggers two downstream actions. You could immediately split the event into two action-specific messages and send them separately. But if those actions need to know about the *original event* (its timing, its user context, its intent), that information is lost. The early projection was lossy. The two handlers now hold fragments, and neither can reconstruct the whole — because the projection has no inverse.

## Heuristics as a symptom of lost context

The most insidious form of this problem is that **the damage is invisible at the point where it hurts**. When you're working in a lower-level module that receives already-projected data, you don't know that relevant semantic information was stripped away upstream. All you see is that the behavior at your level is hard to pin down — there are too many possibilities, the control flow has too many branches, and the logic feels tangled.

This is fundamentally hard to diagnose because the symptom (tangled code, heuristic-based decisions) appears far from the cause (a lossy projection at a distant point upstream). When you're working at the lower level, you can't see what you don't have. The explosion of possibilities is not intrinsic complexity — it's accidental complexity introduced by the projection that stripped away the constraining context.

**The diagnostic clue:** if a module's behavior is hard to specify, if it handles many cases through pattern-matching or heuristics, ask whether it's receiving the right data. Trace the data back upstream. Was there a richer, more concrete form of this data that was projected before it reached here? Could passing the original, unprojected concept have collapsed those many cases into a few obvious ones?

## Diagnostic tests for lossy projection

1. **The reconstruction test.** Is downstream code rebuilding a concept from fragments —  joining tables, querying auxiliary services, or merging data from multiple sources to recover what was once a single value? If so, the projection was lossy relative to what consumers need.

2. **The branch explosion test.** Does one upstream variant produce many downstream branches? When a single `OrderPlaced` event produces different behavior depending on referral source, tier, and time of day, but the billing handler only receives `userId` and `amount`, it must branch on every combination it cannot distinguish.

3. **The heuristic test.** Is code using approximate logic — "we assume this is a referral order if the user signed up within 7 days" —  where the upstream data had an exact answer? The heuristic is a failed attempt to invert the projection.

4. **The relationship test.** Do the parts carry meaning only *together*? If splitting them destroys information that consumers need, keep them together.

5. **The reference preservation test.** Can you project the structure but preserve a link? Sometimes the answer is to split into independent concepts but maintain a reference (an ID, a parent type) that lets consumers reconstruct the relationship when needed. This is a lossy projection with a recovery path —  less convenient than keeping the whole, but not a dead end.

# Remedy

- **Default to keeping concepts together** at the point of origin. Let the data carry its full semantic content through the system. 

- **Project at consumption boundaries, not at origin.** When a specific consumer only needs part of a concept, let that consumer project what it needs — don't pre-project at the source. The source sends the whole; each receiver extracts its slice. This is the dual of "parse at the boundary" from [Making Evidence Explicit](./making-evidence-explicit.md): just as you parse raw input into domain types at the *entry* boundary, you project domain types into view-specific shapes at the *consumption* boundary.

- **If you must project at origin**, verify the projection is sufficient by checking whether any downstream consumer needs to reconstruct the original. If it does, the projection was premature —  either widen it, add an envelope, or move the projection to the consumer side.
