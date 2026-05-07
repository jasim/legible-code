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
  version: "2.2.0"
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

The procedure scales from small subjects (a single file, a short diff) to large ones (whole-codebase audits, sweeping plans) by fanning out evidence-gathering to subagents when a hot spot would otherwise blow context. There is no separate mode to invoke — small subjects run the procedure end-to-end in one mind, large ones dispatch evidence-only probes from within Pass 2. The output shape — 2–5 ranked findings — is the same in both cases.

### Procedure

Work in three passes. Do not start matching diagnostics until Pass 3.

**Pass 1 — Skim (always, orchestrator only).** Read the entire subject end-to-end and build the cross-cutting map: entry points and data flow (code), types and boundaries (code or design), decisions made vs. deferred (plans). Note suspicious regions — long functions, verb-named modules, scattered conventions, string-flag branching, in-place mutation, deep nesting, tangled I/O, unnamed repeated patterns. Skip boilerplate, generated code, and unrelated parts. Produce a short list of **candidate hot spots**. For small subjects this is a deep read; for large subjects it is shallower but still single-pass — directory tour, entry-point inventory, file headers, public symbols, primary types. Do not parallelize this; the cross-cutting map is the foundation everything else rests on, and a sharded skim destroys it.

**Pass 2 — Evidence (orchestrator; fan out when needed).** For each hot spot, gather the concrete evidence you'll need to fire diagnostics — call sites, type signatures, consumer lists, repeated patterns, implicit-vs-explicit contracts. If you can read the relevant code in place without straining context, do that. If gathering evidence for a hot spot would blow context, dispatch a single evidence-only `Explore` probe per hot spot (see **Orchestration** below). Subagents do not diagnose, do not name D-numbers, do not propose remedies, do not rank; they return raw evidence and the orchestrator integrates it.

**Pass 3 — Diagnose (always, orchestrator only).** Walk the diagnostics (see `DIAGNOSTICS.md` for D1–D14) in order over the consolidated evidence. A diagnostic fires only if you can point to specific evidence from the "You have this problem if" bullets — not on vibes. When several fire in the same region, prefer the lowest-numbered (upstream, often causes the others). Rank findings by leverage and emit output.

### Orchestration

How to scale the procedure when the subject is too large for a single mind to hold.

**The load-bearing rule.** *Only the orchestrator names diagnostics.* Subagents gather raw evidence; the orchestrator integrates that evidence and runs the diagnostics against it, using its own Pass 1 skim as the cross-cutting map. This rule exists because several diagnostics are inherently cross-cutting — narrative incoherence (D1) spans entry points to side-effects, missing vocabulary (D6) spans modules, scattered knowledge (D13) is spread by definition, re-litigated invariants (D14) live between a producer and many consumers. A subagent given only a slice of the subject would either over-flag (mistaking local duplication for system-wide scatter) or under-flag (missing the cross-boundary face of the diagnostic). Keeping diagnosis with the orchestrator dissolves the problem instead of mitigating it.

**When to dispatch a probe.** During Pass 1 (skim) or Pass 2 (evidence), if pulling evidence for a hot spot would blow your context — many call sites to enumerate, many consumers to inspect, repeated patterns to confirm across modules — dispatch a single `Explore` subagent for that hot spot. If you can read the relevant code in place without straining context, do that instead. Probes have overhead and lose nuance; reach for one only when the alternative is losing the cross-cutting map. The decision is implicit — it happens in the moment, when you notice that following a hot spot in place would cost you the rest of the subject. There is no separate triage step up front and no threshold to compute.

**Evidence-probe prompt template.** Paste this into an `Explore` call, filling in the hot spot description. The constraints are stated inside the prompt itself, deliberately — they have to bind the subagent, not just the operator briefing it.

> You are an evidence-only worker for a legibility review. Gather raw material about the hot spot below: call sites, type signatures, consumer lists, repeated patterns, implicit-vs-explicit contracts, where data is constructed vs. consumed. Return concrete evidence with `file:line` references.
>
> Constraints — load-bearing:
> - **Do not diagnose.** Do not name diagnostics or D-numbers.
> - **Do not propose remedies.** Do not name notes or refactoring strategies.
> - **Do not rank, prioritize, or judge severity.** Just report what is there.
> - **Do not summarize or interpret.** The orchestrator will integrate the evidence; your interpretation would discard the detail it needs.
>
> Hot spot: `{description — file paths, symbols, the specific question to answer}`
>
> Return a structured list of findings, each with file:line citations. Be concrete and exhaustive within the hot spot's scope; do not stray outside it.

**Concurrency.** Independent probes can run in parallel. Cap at 4–6 in flight as a starting point — revisit after real use. Sequential probes are fine when later ones depend on earlier results (e.g. enumerate consumers first, then probe each consumer cluster).

### Scope Rules

- **Diff or PR:** Weight findings toward issues the diff introduces or worsens. Don't pile on pre-existing problems unless they make the diff materially harder to review.
- **Plan or design:** Apply diagnostics to the proposed structure — which decisions are reified, where the seams are, what vocabulary is introduced, what knowledge would be scattered. Flag problems the plan would *bake in* if implemented as-written.
- **Files with no specific question:** Use the user's framing as the signal ("hard to test" → D3/D4/D5; "hard to change" → D7/D8/D13; "hard to follow" → D1/D2/D11).
- **Nothing fires:** "No high-leverage legibility issues found; here's what I checked" is a valid output. Do not invent problems.

---

## Diagnostics

The fourteen recurring legibility problems live in [`DIAGNOSTICS.md`](DIAGNOSTICS.md). Their conceptual reasoning lives in [`INTRODUCTION.md`](INTRODUCTION.md). Each diagnostic points to a remedy in `notes/*.md`. Read the matching note before recommending — do not paraphrase from memory.
