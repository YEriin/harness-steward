---
name: debug-with-history
description: Use when a bug, error, unexpected behavior, or production incident is encountered — frames the problem, searches the project's lessons-log for similar root causes, checks if the bug is a manifestation of a declared architectural boundary violation, and guides toward root cause (not symptom patch) before the user enters their debugging methodology (e.g. superpowers:systematic-debugging if installed, or their own process). Triggers include "there's a bug", "production issue", "this isn't working", "users are reporting X", "error at Y", "fix this", "why is this failing".
---

# Debug With History

Project-aware debug entry point. When a bug surfaces, check whether the project has encountered something similar before (lessons-log) and whether the symptom is a fingerprint of a known boundary violation (architectural-intent) — before jumping into methodology. Hand off to the user's debugging workflow (e.g. `superpowers:systematic-debugging` if that plugin is installed, or their own process) for the actual investigation.

## Core principle

**Most bugs have fingerprints.** If the project has been alive for any meaningful time, similar bugs have likely been hit before; the response went into `.harness/lessons-log.md`. Starting debugging from scratch without consulting that history means repeating someone else's investigation. Starting without checking architectural intent means missing the class of bugs that surface at declared boundary-violation points.

Read-only consultation. This skill does not debug, does not fix, does not modify sediment. It frames the problem, loads relevant history, and hands off.

## Prerequisites

- `.harness/` exists (if not → route to `bootstrap-new-project` or `adopt-into-existing-project`)
- A concrete bug / error / unexpected behavior to frame (not "something's slow" — need specifics)

## When to use / not use

**Use when:**
- User reports a bug, production incident, regression, or unexpected behavior
- User asks *"why is this failing?"* or *"what's wrong with X?"*
- A test is failing in a way that isn't obvious
- About to enter debug mode and the project has `.harness/` sediment worth consulting

**Do NOT use for:**
- Trivial bugs (typo, clearly-obvious fix, literal error pointing at the line)
- Already mid-investigation with methodology loaded — this is front-end, not mid-stream
- Pure performance tuning or optimization (different activity)
- Feature request / requirement — use `evaluate-requirement` instead
- No `.harness/` (route to bootstrap/adopt first; without sediment, proceed with the user's debugging workflow directly — there's no project memory to consult)

## Phase 0 — Frame the problem

Pin down the bug before loading context:

1. **Symptom** — what exactly is observed? (error message, wrong output, timeout, crash)
2. **Trigger** — when does it occur? (always / specific conditions / flaky)
3. **Reproducibility** — can it be reproduced? How?
4. **Blast radius** — one user / all users / data corruption / degraded perf / external impact?
5. **User's current hypothesis** — capture it, but **don't let it anchor** the next phases; anchoring early closes off the history search

If any of (1)-(3) is unclear after one question round, **stop and ask**. A vague bug loaded against vague history produces noise.

## Phase 1 — Historical pattern search

Grep `.harness/lessons-log.md` for:

### 1a. Symptom match
Search for entries with similar `**What:**` descriptions — same error class, same module, same trigger pattern.

### 1b. Root-cause match
Search for entries with similar `**Why:**` root causes — the underlying class of problem, not the surface symptom.

### 1c. Module match
For each file/module implicated in Phase 0, grep for that path. Surface any entries touching the same code path, even if the prior issue was different.

### 1d. Recent activity
List the last 3-5 lessons-log entries regardless of match, for context on what the project has been dealing with lately.

Surface findings by relevance:
- **🔴 Strong match** — same root cause AND same module: this may be a recurrence (category-C promotion candidate — flag for `extract-lesson` after fix lands)
- **🟡 Partial match** — same module or same root cause, not both
- **🟢 Context-only** — recent activity in adjacent code, informational
- **No match** — brand-new bug class for this project; note explicitly

## Phase 2 — Architectural-intent boundary check

Read `.harness/architectural-intent.md`. Two checks:

### 2a. Boundary-manifestation check
For each *"does not know about"* constraint in Key Abstractions, check:
- Does the bug trigger cross this constraint?
- Does the call stack or error path traverse a module that isn't supposed to know about the module it's calling?
- Example: intent says *"Job does not know about HTTP"*; bug is an HTTP timeout surfacing in `jobs/scheduler.ts` → flag **INTENT-VIOLATION-MANIFESTATION**: the bug may be a symptom of the declared coupling constraint being violated in code

### 2b. IS-NOT manifestation
Does the bug's symptom match something the system is explicitly NOT supposed to do?
- Example: intent says *"Not a real-time system; eventual consistency is fine"*; bug report is *"data takes 2 seconds to update; users complaining"* → flag **BOUNDARY-VIOLATION-MANIFESTATION**: the bug may be a requirements mismatch, not a code defect

Not all bugs trigger these flags — that's expected. When they do, they redirect the investigation profoundly. The `-MANIFESTATION` suffix indicates dynamic/symptomatic signal (vs. the static `INTENT-VIOLATION` / `BOUNDARY-VIOLATION` from review-task/evaluate-requirement which check code/requirements directly).

## Phase 3 — Root-cause direction

Based on Phases 1-2, produce a brief debugging orientation:

### 3a. Likely category
Which kind of bug is this, based on history + intent?
- **Recurrence** — same root cause as prior entry → prior fix may be incomplete or bypassed; check what changed
- **Boundary breach** — Phase 2 flagged intent-violation-manifestation → fix likely requires restoring boundary, not just patching symptom
- **Known-class** — similar module history but different root cause → prior investigations may have clues
- **Novel** — no history match → start from first principles; note for future lesson capture

### 3b. Anti-band-aid reminder
Surface the relevant CLAUDE.md rule explicitly: *"When fixing a bug, trace to root cause before applying a fix. Catch-and-ignore, suppressing warnings, or deleting failing assertions to 'make it pass' is hiding, not fixing."*

If the user seems to want a quick symptomatic patch, explicitly ask: *"This is a surface patch — do you want to ship it while the real fix is being worked, or trace root cause first?"* The answer shapes what comes next.

### 3c. Quick-fix vs. real-fix cost comparison
Where applicable, estimate:
- Symptom patch: surface fix, ships fast, may mask real issue
- Root-cause fix: addresses underlying cause, may be more work
- User chooses, but the cost comparison should be explicit

## Phase 4 — Hand off

End the skill. Name the next action:

- **If ready to investigate methodically** → enter the user's debugging workflow (`superpowers:systematic-debugging` if installed, or their own)
- **If root cause is already clear from Phases 1-2** → hand off to fix phase directly, then `review-task` post-fix, then `extract-lesson` if lesson-worthy
- **If strong recurrence match from Phase 1a** → after fix, invoke `extract-lesson` to evaluate category-C promotion of this pattern

## Output format

```
## Debug Briefing — <one-line symptom>

### Framed
- Symptom: <exact observation>
- Trigger: <conditions>
- Reproducibility: <how to reproduce, or "flaky / intermittent">
- Blast radius: <scope>
- User hypothesis: <captured, not anchored>

### Historical matches (from .harness/lessons-log.md)
- 🔴 Strong match (recurrence): <date> <summary>
- 🟡 Partial match: <date> <summary>
- 🟢 Context: recent entries involve <brief>
- (or "No matches in lessons-log.")

### Intent-boundary check
- 🔴 INTENT-VIOLATION-MANIFESTATION: <which constraint, how the bug traverses it>
- (or "No boundary-manifestation detected.")

### Direction
- Likely category: <recurrence / boundary-breach / known-class / novel>
- Quick-fix cost: <estimate>
- Real-fix cost: <estimate>

### Next action
Hand off to the user's debugging workflow (e.g. `superpowers:systematic-debugging` if installed), OR
if recurrence confirmed: fix → `review-task` → `extract-lesson` (category-C promotion candidate)

Reminder: CLAUDE.md's Behavioral Discipline rule — trace root cause before patching; no catch-and-ignore.
```

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Debugging inside this skill | Hand off to a debugging workflow (e.g. `superpowers:systematic-debugging` or the user's own process). This skill is orientation, not investigation. |
| Letting user's hypothesis anchor the search | Capture the hypothesis in Phase 0 but run Phases 1-2 independently. History may reveal a different cause. |
| Skipping Phase 0 framing | Vague bugs + vague history = noise. Pin down the bug first. |
| Ignoring architectural-intent boundary check | Many long-lived bugs are manifestations of coupling the project said shouldn't exist. Always do Phase 2. |
| Recommending a symptomatic fix without noting the cost | User has the right to ship fast, but they need to SEE the tradeoff. |
| Auto-promoting to category-C after single match | Promotion is `extract-lesson`'s decision based on its tree. Just flag as candidate. |

## Relationship to other skills

- **Preceded by** (rare): `evaluate-requirement` if the "bug" turns out to be a requirements mismatch rather than a code defect
- **Followed by**: the user's debugging workflow (e.g. `superpowers:systematic-debugging` if installed, or their own process)
- **After the fix**: `review-task` on the diff, then potentially `extract-lesson` if a lesson emerged (especially if Phase 1 flagged recurrence)
- **Complements** external debugging methodologies — this skill loads project-specific context before methodology begins
- **Distinct from** `evaluate-requirement` (that handles new feature requests, not bugs)

## One-sentence summary

**Before debugging methodology starts, frame the bug and consult the project's sediment — lessons-log for precedent, architectural-intent for boundary-manifestation — then hand off to the user's debugging workflow with context pre-loaded.**
