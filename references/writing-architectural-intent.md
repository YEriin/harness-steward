# Writing Architectural Intent

Before any harness can work, the project needs an **architectural intent** — a short, durable document stating what the system is, what it isn't, and where it's heading.

This document is **Layer 0** of AI coding discipline: harness enforces structure; architectural intent defines *what structure to enforce*. Without it, harness executes cargo-cult rules.

## Why this must be written by you

**AI cannot write this well for you.** Architectural intent is judgment:

- Boundaries you care about
- Abstractions you've chosen (and rejected)
- Directions you're moving in (and directions you're refusing)

Those come from you. AI can help you articulate it once you have the thoughts, but it cannot originate them.

Without this doc:

- Every new feature tempts the AI in a slightly different architectural direction
- `CLAUDE.md` rules become tactical ("use this util") instead of strategic ("this module doesn't do persistence")
- Macro-loop audits have nothing to measure drift **against**
- `extract-lesson` has no yardstick for judging whether a lesson contradicts your architectural direction

## What goes in it

**Four sections. 1-2 pages total. Long is worse than short here.**

### 1. What this system IS

2-3 sentences. The central purpose, as you'd explain to a new engineer on Day 1.

### 2. What this system is NOT

A hard boundary list. Examples:

- *"Not a data warehouse — we do ephemeral processing only"*
- *"Not multi-tenant — one instance, one customer's data"*
- *"Not a real-time system — eventual consistency is acceptable"*

**The negatives do more work than the positives.** They prevent scope creep and architectural drift in specific directions. Most scope creep shows up as "it would be nice to also do X" — the negative list makes the pushback explicit and authoritative.

### 3. Key abstractions

The 3-5 core concepts the code is organized around. For each:

- Name and one-sentence purpose
- Which module owns it
- What it's allowed to know about / not know about

Example:

```
- User     — authenticated identity. Lives in `auth/`. Does not know about billing.
- Session  — user's current interaction context. Lives in `session/`. Does not know about persistence backend.
- Job      — async task unit. Lives in `jobs/`. Does not know about HTTP / responses.
```

The "does not know about" half is critical. It defines the coupling you refuse to allow.

### 4. Evolution direction

Where this is heading in 3-6 months. Not a roadmap — a **direction of travel**. Examples:

- *"We'll add streaming responses but not streaming ingestion."*
- *"We'll split auth into a separate service; code should anticipate this."*
- *"We won't support on-prem deploys; cloud-only."*

This section gives AI (and humans) a coherent sense of which way to lean when decisions are ambiguous.

## How it gets used

Harness skills refer to this document at specific moments:

- `bootstrap-new-project` — helps you write it Day 0
- `audit-harness` — checks if recent `CLAUDE.md` rules align with stated intent
- `audit-repo-hygiene` — reports if the codebase structure has drifted from declared module ownership
- `extract-lesson` — tests whether a candidate lesson is consistent with the intent

If no skill reads your architectural intent, it's a dead document. At least one periodic process must check code against it.

## Maintenance cadence

**This doc changes slowly — quarterly at most.**

If you find yourself editing it frequently, either the project is genuinely pivoting (fine — update it) or you're using it as a scratchpad for evolving thinking (not fine — move that elsewhere, probably `.harness/lessons-log.md`).

Update triggers:

- Major pivot in what the system does
- Retiring a key abstraction or adding one
- Consciously changing the negative boundaries

## Failure modes

- **Too long** — if it's 10 pages, no one reads it, including you. Cap at 2 pages. Cut ruthlessly.
- **Too aspirational** — *"We will have a beautiful architecture"* is vapor. State what IS, not what you hope.
- **Too implementation-detailed** — this isn't a design doc. Don't put API endpoints, schemas, or code samples here.
- **Never referenced** — if no skill or audit reads it, it's dead. Fix by wiring it into your macro loop.
- **Written once, never revisited** — the opposite failure. If it hasn't been touched in 9 months and the project has evolved, it's stale and misleading.

## Template location

A fillable skeleton lives at `templates/architectural-intent.template.md`. **Don't copy it unchanged** — every section must be filled with your project's reality. An unfilled template is worse than no template.
