---
name: scope-implementation
description: Use when about to start implementing a feature, change, or refactor — loads project-specific context (architectural boundaries, past lessons, existing conventions) before implementation begins, so the implementation doesn't silently violate boundaries or repeat past mistakes. Triggers include "let's implement X", "how should I build Y", "start coding", "begin the feature", "what's the approach for Z". Hands off to the user's planning / implementation workflow (e.g. superpowers:writing-plans or superpowers:test-driven-development if installed, or the user's own methodology).
---

# Scope Implementation

Pre-implementation project briefing. Loads relevant architectural intent, historical lessons, and existing conventions into the conversation so implementation decisions are informed by project reality rather than generic best practice.

## Core principle

**The expensive architecture mistakes happen in the first 10 minutes of implementation.** AI starts writing, picks a structure that seems reasonable in isolation, and 200 lines later the decision is baked in. This skill front-loads the project context that prevents those early missteps — before any code is written.

Report-and-brief only. Does not plan (that's the next step, handled by your planning workflow — e.g. `superpowers:writing-plans` or your own). Does not implement. Does not edit sediment or code.

## Prerequisites

- `.harness/` exists (if not → route to `bootstrap-new-project` or `adopt-into-existing-project`)
- User has a concrete implementation target (not just "a new feature" — at least a one-sentence description)
- The change is non-trivial (skip for single-line tweaks, typo fixes, or formatting)

## When to use / not use

**Use when:**
- User says *"let's implement X"*, *"start coding"*, *"begin the feature"*
- User asks *"how should I approach implementing Y?"*
- About to enter a planning / implementation workflow and the project has `.harness/` sediment worth consulting first
- About to modify a module with meaningful architectural constraints

**Do NOT use for:**
- Target is fuzzy / still being scoped — use `evaluate-requirement` first
- Trivial changes (typo, formatting, single-line fix) — overhead exceeds value
- Already mid-implementation — this is a front-load skill, not mid-stream
- Tasks without project context to load (no `.harness/`, or fresh repo) — no signal to surface

## Phase 0 — Confirm the target

Before loading anything, pin down:

1. **One-sentence target**: what is being implemented? (Ask if the user hasn't stated it clearly.)
2. **Affected modules/files**: which parts of the code will likely change? Ask the user to name them, or propose based on the target and let the user confirm.
3. **Success criteria**: how will we know this is done? (Ask if vague — vague targets produce vague briefings.)

If any of these remain fuzzy after one round of questions, **stop and route to `evaluate-requirement`** — that skill handles requirement scrutiny; this one assumes the target is clear enough to scope implementation.

## Phase 1 — Load architectural boundaries

Read `.harness/architectural-intent.md`. For each affected module identified in Phase 0:

### 1a. Relevant key abstractions
Find the module in architectural-intent's Key Abstractions list. Surface:
- What this module IS (one-sentence purpose)
- What it is allowed to know about
- What it is explicitly **not** allowed to know about ("does not know about Y")

### 1b. System-level boundaries
Re-read architectural-intent's "IS NOT" section. Any of those boundaries apply to this implementation?
- Example: intent says *"Not a queue — no durability guarantees"*; implementation plans to add a retry queue → surface as a boundary concern

### 1c. Evolution-direction alignment
Read architectural-intent's Evolution Direction. Is this implementation:
- Aligned with stated direction? — note it
- Pushing against it? — surface as a concern (possibly DIRECTION-CONFLICT); user decides whether to proceed, update intent, or reconsider scope

## Phase 2 — Consult past lessons

Read `.harness/lessons-log.md` (lessons-log is append-only; this is a read-only consultation). Two searches:

### 2a. Module-level history
For each affected file/module path from Phase 0, grep for that path in lessons-log. Surface matches: *"This module has history: see lesson 2026-02-14 — retry without backoff caused cascading failures."*

### 2b. Feature-type patterns
For the feature type (auth, payment, upload, migration, etc.), grep lessons-log for that domain. Surface relevant prior lessons, especially those marked with structural responses (category A) — past structural defenses often indicate design constraints this implementation should respect.

If multiple matches, prioritize: (1) entries with "Related:" cross-references (pattern indicators), (2) recent entries, (3) matches on both module AND feature type.

## Phase 3 — Identify existing conventions

For each affected module, scan the existing code for conventions that the new implementation should match (per `CLAUDE.md`'s "Read surrounding code; match existing patterns"):

Using Glob and Grep:
- **Naming**: how are similar things named here? (functions, classes, files)
- **Error handling**: try/catch style? error objects? thrown vs returned? middleware-handled?
- **Layering**: does this module have a consistent layer pattern (route → handler → service → repository)?
- **Testing**: where are tests, how are they structured, what fixtures/factories exist?
- **Imports**: local import style, module resolution conventions

Produce a short "conventions-in-this-module" summary. Do NOT sample code broadly — focus on the files closest to the affected area (same directory, same module).

## Phase 4 — Compile the briefing

Produce a concise Implementation Briefing in this format:

```
## Implementation Briefing — <one-sentence target>

### Affected modules
- <module path 1> — <role in this change>
- <module path 2> — <role in this change>

### Architectural constraints (from .harness/architectural-intent.md)
- <module> must not know about <forbidden coupling>
- System is NOT <boundary from IS-NOT section> — relevant here because <reason>
- Evolution direction: <aligned / neutral / conflicts with "<direction>">

### Past lessons (from .harness/lessons-log.md)
- <YYYY-MM-DD>: <one-line lesson + response>
- <YYYY-MM-DD>: <one-line lesson + response>
- (or "No directly relevant entries in lessons-log.")

### Existing conventions (from code scan)
- Naming: <observed pattern>
- Error handling: <observed pattern>
- Layering: <observed pattern>
- Tests: <where / how>

### Risk flags
Use severity markers when multiple risks exist:
- 🔴 <high-signal risk: e.g. INTENT-VIOLATION, DIRECTION-CONFLICT — resolve before implementation>
- 🟡 <medium-signal risk: e.g. UNDECLARED-ABSTRACTION, pattern deviation — decide in plan>
- 🟢 <low-signal risk: informational historical match>
- (or "None identified from Phases 1-3.")

### Next action
Hand off to the user's implementation methodology. Typical options:
- A structured planning step (`superpowers:writing-plans` if that plugin is installed, or your own planning workflow)
- TDD-driven implementation (`superpowers:test-driven-development` if installed, or your own test-first workflow)
- Direct implementation with the briefing in hand

This briefing will carry as context into whichever path you choose.
```

## Phase 5 — Hand off (do NOT implement here)

End the skill here. Explicitly name the next action — a planning / implementation skill from another bundle, the user's own workflow, or invoking another harness-steward skill like `debug-with-history` if the target turns out to be a bug fix.

**Do NOT write a plan.** Planning is the next step, not this one.
**Do NOT write code.** Implementation is the next step, not this one.
**Do NOT auto-invoke anything.** Let the user choose the methodology.

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Skipping the briefing and going straight to code | The whole point is pre-loading context; skipping defeats the skill |
| Writing the plan yourself | Hand off to the user's planning step (`superpowers:writing-plans` if installed, or their own). Don't merge the two. |
| Running on trivial changes | Skip for < 10 lines, typo fixes, comment-only — briefing overhead exceeds value |
| Inventing architectural constraints not in intent | Only surface what's actually written in architectural-intent.md. If user thinks a constraint is missing, route to update that doc before implementation. |
| Producing a 5-page briefing | Keep it scannable — one screen. If the briefing is longer than the expected implementation plan, it's bloat. |
| Running when .harness/ is empty / bootstrap never happened | No signal to load. Route to bootstrap-new-project or adopt-into-existing-project first. |

## Relationship to other skills

- **Preceded by**: `evaluate-requirement` (if target is still fuzzy)
- **Followed by**: the user's methodology for planning / implementation (e.g. `superpowers:writing-plans` or `superpowers:test-driven-development` if installed)
- **Eventually followed by**: `review-task` (after implementation completes)
- **Complements** `bootstrap-new-project` / `adopt-into-existing-project` (those created the sediment this skill consumes)
- **Distinct from** creative-scoping skills (e.g. `superpowers:brainstorming`); this skill starts after the idea is settled

## One-sentence summary

**Before implementation starts, load architectural intent + past lessons + existing conventions into the conversation — 1 screen max — then hand off to methodology skills with project reality pre-loaded.**
