---
name: evaluate-requirement
description: Use when a new requirement, feature request, or PRD is being considered — evaluates fit against architectural intent, surfaces historical precedent, and produces decision-support output (GO / MODIFY / REJECT / DEFER) before implementation work begins. Triggers include "should we add X", "new feature", "write a PRD", "consider adding", "we need Y", "can we support Z", "new requirement". Distinct from scope-implementation (which assumes requirement is settled and loads context for building) and superpowers:brainstorming (which is creative exploration without project context).
---

# Evaluate Requirement

Pre-scope requirement evaluation. When a new feature / change / PRD is proposed, check it against the project's stated architectural intent and past history before any implementation effort starts. Surface decision support — do not make the decision for the user.

## Core principle

**The cheapest requirement to reject is the one rejected before anyone writes a plan.** Once implementation starts, sunk cost creates pressure to ship even requirements that shouldn't exist. This skill operates earlier in the lifecycle, when rejection, modification, or deferral costs nothing but a conversation.

Report-and-advise only. This skill does not implement, does not plan, does not modify `.harness/architectural-intent.md` or `.harness/lessons-log.md`.

## Prerequisites

- `.harness/` exists (if not → route to `bootstrap-new-project` or `adopt-into-existing-project`)
- A concrete requirement to evaluate — at minimum a one-paragraph description of what is being proposed and why

## When to use / not use

**Use when:**
- User proposes a new feature or capability
- PRD / spec / requirement is being considered
- User asks *"should we add X?"* or *"can we support Y?"*
- A stakeholder request has arrived that hasn't been scoped yet
- Tech-debt work is proposed (still a requirement — check against intent)

**Do NOT use for:**
- Bug fixes — bugs don't need requirement evaluation (use `debug-with-history`)
- Trivial changes (dependency bump, typo fix, small refactor) — overhead exceeds value
- Requirement already decided and being scoped — use `scope-implementation`
- Requirement already being implemented — this is too-late-in-lifecycle
- Pure creative brainstorming with no concrete ask yet — first use whatever creative-exploration workflow you prefer (e.g. `superpowers:brainstorming` if that plugin is installed), then come back here when a concrete proposal emerges

## Phase 0 — Restate the requirement

Pin down what is being proposed, in the user's words. Answer:

1. **What** is being requested? (one paragraph, concrete)
2. **Why** — the driver: business need, user story, performance concern, bug class, legal, cost, other?
3. **Success criteria** — how will we know this works once shipped?

If any of the three is unclear after one round of questions, **stop and ask**. A vague requirement evaluated produces a vague evaluation, which is worse than no evaluation.

## Phase 1 — Architectural-intent fit

Read `.harness/architectural-intent.md`. Four checks against stated intent.

> **Note on tag naming vs review-task/scope-implementation**: this skill uses `BOUNDARY-VIOLATION` and `ABSTRACTION-GAP` at **requirement time** — *"this proposal conflicts with intent; decide before building"*. `review-task` uses `INTENT-VIOLATION` and scope-implementation uses `UNDECLARED-ABSTRACTION` at **code time** — *"this code has already crossed the line; decide what to do now"*. Same conceptual family, deliberately distinct names to preserve the lifecycle-moment signal in the finding.

### 1a. Alignment with what the system IS
Does this requirement extend the stated purpose, or push into new territory? If new territory → not automatically wrong, but flag **SCOPE-EXPANSION** — the system's purpose may need updating before/after.

### 1b. Collision with what the system IS NOT
Read the IS-NOT boundary list. Does this requirement cross any boundary?
- Example: intent says *"Not a storage layer; never writes events to disk"*; requirement asks for event replay from disk → flag **BOUNDARY-VIOLATION**

This is the strongest signal. A BOUNDARY-VIOLATION is usually a REJECT signal (or requires a conscious decision to update intent).

### 1c. Key-abstraction impact
Does this requirement fit within existing key abstractions, or require introducing new ones?
- If fits → low risk, proceed
- If requires new abstraction not in intent → flag **ABSTRACTION-GAP**: architectural intent must be updated before/as-part-of this work

### 1d. Evolution-direction alignment
Does this push in the direction intent says we're moving, against it, or orthogonal?
- Aligned → tailwind
- Orthogonal → neutral
- Against → flag **DIRECTION-CONFLICT** — user decides whether to proceed (and update intent) or reconsider

## Phase 2 — Historical precedent

Search `.harness/lessons-log.md` (read-only). Grep for:

- **Similar requirements**: same domain, similar user story, similar feature class (e.g., "retry", "rate limit", "webhook")
- **Similar rejections / deferrals**: past work that didn't land, or similar requirements that were rejected previously
- **Related pain**: past bugs or incidents in the same module/domain

Surface findings by relevance:
- **🟢 Favorable precedent** — "we've built similar before; lesson 2026-01-10 shows it worked"
- **🟡 Mixed precedent** — "we tried similar; see lesson 2026-02-14 for the gotchas"
- **🔴 Unfavorable precedent** — "we rejected similar in lesson 2026-03-01, because..."
- **No precedent** — brand-new territory; note this so the user knows there's no safety net

## Phase 3 — Scope and risk rough-sizing

Based on Phases 1-2, estimate:

### 3a. Rough effort scale
One of: **hours / days / weeks / month+**. This is a sanity check, not a commitment. Purpose: catch requirements that sound small but aren't (or vice versa).

### 3b. Risk factors
List applicable risks from this short taxonomy:
- **Security** — auth, secrets, data exposure, injection
- **Data integrity** — migrations, schema changes, irreversible writes
- **Performance** — hot-path additions, N+1 patterns, unbounded growth
- **Backward compatibility** — API contract changes, breaking consumers
- **Operational** — new infra, new dependency, new on-call surface

### 3c. Unknowns worth prototyping
What assumptions would a prototype validate? If prototyping is cheap and assumptions are load-bearing, flag for prototype-first.

## Phase 4 — Decision support

Compile output in this format:

```
## Requirement Evaluation — <one-line restatement>

### Restated
- What: <one paragraph>
- Why: <driver>
- Success: <criteria>

### Intent fit
- Alignment: <aligned / scope-expansion / boundary-violation / direction-conflict>
- Key-abstraction impact: <fits existing / abstraction-gap>
- Notes: <specific constraints from intent that apply>

### Historical precedent
- 🟢 / 🟡 / 🔴 / none
- Relevant entries: <list, with dates>

### Scope & risk
- Rough scale: <hours / days / weeks / month+>
- Risk factors: <list from taxonomy>
- Prototype first: <yes/no + what assumption>

### Recommendation
One of:

**GO** — evaluation is clean; ready to hand off to scope-implementation.
**MODIFY** — requirement needs refinement; specific changes needed: <list>.
**DEFER** — depends on <X> landing first, or needs prototype.
**REJECT** — conflicts with <specific intent boundary or historical lesson>; surface the conflict and wait for user decision.

This recommendation is a signal, not a verdict. User decides.

### Next action
If GO or MODIFY-accepted: hand off to `scope-implementation` to load implementation context.
If DEFER: note what's being waited on; record in current conversation or project notes.
If REJECT: if user agrees, done. If user overrides, note the architectural intent needs updating to reflect the new direction.
```

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Making the decision for the user | Surface options + evidence. User decides. |
| Skipping Phase 0 (restating) | Without restating, you're evaluating your *impression* of the requirement. Restate explicitly. |
| Inventing architectural constraints not in intent | Only use what's actually in `architectural-intent.md`. If user wants a constraint enforced that isn't there, route to updating intent first. |
| Writing the implementation plan | Out of scope. Hand off to `scope-implementation` next (then to the user's planning workflow — e.g. `superpowers:writing-plans` if installed — after that). |
| Evaluating vague requirements | Requirements that are vague produce vague evaluations. Ask one round of clarifying questions; if still vague, stop. |
| Auto-rejecting on any flag | Multiple requirements are worth breaking old boundaries — that's evolution. Flags are signals; the user weighs them. |

## Relationship to other skills

- **Preceded by**: any creative-exploration workflow (e.g. `superpowers:brainstorming` if installed) when the requirement itself needs shaping before evaluation
- **Followed by**: `scope-implementation` (if GO decision) — loads implementation context
- **Precedes** the user's planning workflow (e.g. `superpowers:writing-plans` if installed) — planning structures the work once GO is decided; this evaluates whether GO is appropriate in the first place
- **Distinct from** `scope-implementation` (which assumes the requirement is settled)
- **Distinct from** `review-task` (which checks completed work, not proposed work)

## One-sentence summary

**Before anyone plans or builds, check the proposed requirement against architectural intent + past history; produce GO / MODIFY / DEFER / REJECT decision support — the user decides.**
