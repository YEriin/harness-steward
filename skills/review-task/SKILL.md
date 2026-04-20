---
name: review-task
description: Use when a coding task (feature, bug fix, refactor) is complete and about to be handed off or committed, to review the change against project-specific sediment (architectural intent, lessons-log, CLAUDE.md rules) before issues accumulate. Triggers include "done, ready to commit", "finished implementation", "fix complete", "task complete", or any moment AI is about to claim a coding change is finished. Distinct from periodic audit-harness / audit-repo-hygiene (which scan the whole project on cadence); this skill reviews only the current diff, every task.
---

# Review Task

Per-task harness review. Catches project-context violations at the moment of creation, before they accumulate into the kind of decay that periodic audits surface too late.

## Core principle

**Problems are cheapest to fix the second they're written.** Once a bad change is committed, subsequent work depends on it, other people read it, and the cost of fixing multiplies. Periodic audits are a safety net; this skill is the front-line filter.

This skill is **report-only**. It does not auto-fix, does not block commits, does not edit code. It surfaces findings; the user (and the AI via subsequent rounds) decides what to act on.

## Prerequisites

- `.harness/` exists (if not → route to `bootstrap-new-project` or `adopt-into-existing-project`)
- A coding task has just been completed — there is a diff to review (either uncommitted in working tree, or recently committed but not yet handed off)
- The change is non-trivial (skip for single-line typo fixes or comment-only changes)

## When to use / not use

**Use when:**
- AI just finished implementing a feature
- AI just completed a bug fix
- AI just finished a refactor
- User is about to commit / hand off a task
- Any moment of "done, next?"

**Do NOT use for:**
- In-progress work — wait until the task is actually complete
- Trivial changes (typo fixes, comment-only edits, formatting-only) — overhead exceeds value
- Full-repo concerns — that's `audit-repo-hygiene` (scans everything, different scope)
- Harness config review — that's `audit-harness`

## Phase 0 — Scope the review

Read the diff:

- Prefer `git diff HEAD` (includes staged + unstaged) if the change is uncommitted
- If already committed, `git diff HEAD~1 HEAD` for the latest commit, or the commit range the user specifies

Identify:
- **Files touched** — group by module/directory
- **Lines changed** — scale indicator; if very small (< 10 lines), this skill may be overkill
- **Stated task scope** — what the user/AI said this task was about (from conversation context)

If no clear task scope, ask the user for a one-sentence summary before proceeding.

## Phase 1 — Check against architectural intent

Read `.harness/architectural-intent.md`. For the modules touched by the diff:

### 1a. Module ownership violations
For each Key Abstraction in architectural-intent, look at the "does not know about" clause:
- Did the diff introduce an import, call, or reference that crosses a stated boundary?
- Example: architectural-intent says *"Job does not know about HTTP"*; diff imports from `http/` inside `jobs/scheduler.ts` → flag **INTENT-VIOLATION**

### 1b. New abstractions not in intent
Did the diff introduce a new top-level concept (new module, new exported class, new cross-cutting pattern) that isn't named in architectural-intent's key-abstractions list?
- Not automatically wrong, but significant enough to flag **UNDECLARED-ABSTRACTION** — user decides whether to update intent or reconsider
- This is the Architecture Ownership rule (from `CLAUDE.md`) activating: architecture decisions belong to the user

### 1c. Direction-of-travel conflict
Did the change push the project in a direction explicitly rejected by intent's Evolution section?
- Example: intent says *"We explicitly reject persistence features"*; diff adds a database dependency → flag **DIRECTION-CONFLICT**

## Phase 2 — Check against CLAUDE.md rules (diff-level)

These checks apply per-task (not repo-wide as in `audit-repo-hygiene`):

### 2a. Drive-by changes
Does the diff include changes to files/modules unrelated to the stated task?
- Cross-reference task scope (Phase 0) with files touched
- Flag **DRIVE-BY** for unrelated changes (tightens "Only do what was asked" rule)

### 2b. Evidence attachment
If the AI claimed "tests pass", "fix verified", "works" — did the user see verification output in the current session?
- Search conversation for command output matching the claim
- If claim was made without visible evidence → flag **NO-EVIDENCE** (tightens Behavioral Discipline)

### 2c. Destructive operations slipped through
Did the diff include destructive operations (file deletions, migrations, dependency removal) without explicit prior user approval?
- Grep the diff for mass deletions, removed dependencies in manifest
- Flag **UNCONFIRMED-DESTRUCTIVE** (tightens Destructive Operations rule)

### 2d. Root-cause vs band-aid
If the task was a bug fix, does the change address the root cause or suppress the symptom?
- Signals of band-aid: empty catch blocks added, warnings suppressed, assertions deleted, try/except around things that should not fail
- Flag **POSSIBLE-BANDAID** with evidence from the diff

### 2e. Comments discipline (if applicable)
Are there new comments in the diff explaining WHAT the code does (redundant) or narrating the task ("added for X ticket")?
- Flag **NARRATIVE-COMMENT** — low signal but adds up

## Phase 3 — Check against lessons-log

Search `.harness/lessons-log.md` for entries relevant to the diff:

### 3a. Files/modules in history
For each file/module touched, grep lessons-log for that path. If any hits:
- Surface the matching entries as context — "This module has history: see lesson 2026-03-14"
- Not a finding; informational. But useful to decide whether current change respects prior learning.

### 3b. Pattern match
If the task was a bug fix, grep lessons-log for similar root-cause phrasing. If matches:
- If this is a **recurrence of a past issue** that was logged-only, flag **RECURRENCE-DETECTED** — candidate for category-C promotion via `extract-lesson`

## Phase 4 — Identify lesson candidates

Based on findings and the task itself, suggest what might be worth routing to `extract-lesson`:

- **Category A candidate** — if a structural fix (hook, lint, type, test) could have prevented an issue surfaced here
- **Category B candidate** — if the task revealed something about the system that architectural-intent should capture (e.g., a new boundary became clear)
- **Category C candidate** — if Phase 3b flagged recurrence
- **Category E candidate** — if the task hit something mildly noteworthy but not rule-worthy

Do NOT invoke `extract-lesson` directly — just flag candidates. User chooses whether to run it next.

## Output format

```
## Task Review — <one-line task summary>
**Date:** <YYYY-MM-DD>
**Scope:** <N> files, ~<N> lines | Task: <stated scope from Phase 0>

### 🔴 High signal (resolve before commit)
- [INTENT-VIOLATION] src/jobs/worker.ts:42 imports from src/http/ — architectural-intent says Job does not know about HTTP
- [UNCONFIRMED-DESTRUCTIVE] package.json removed `left-pad` dependency; no prior user approval in this session

### 🟡 Medium signal (review)
- [DRIVE-BY] src/billing/invoice.ts changed but task was "fix auth redirect loop"
- [NO-EVIDENCE] "tests pass" claim made without visible test output this session
- [POSSIBLE-BANDAID] src/api/fetch.ts:88 added try/catch that swallows error silently; root cause unclear
- [RECURRENCE-DETECTED] src/api/payment.ts timeout bug similar to lesson 2026-02-14 ("retry without backoff") — category-C promotion candidate

### 🟢 Low signal (informational)
- [HISTORY-MATCH] src/api/fetch.ts was touched by lesson 2026-02-14 (timeout handling)
- [NARRATIVE-COMMENT] src/auth/login.ts:12 comment "// added for ticket AUTH-42" — reword or remove

### Lesson candidates
- **Category A (structural)**: three endpoints added direct retry loops; propose a lint rule banning direct retry (forcing use of the project's shared http client with built-in backoff) — structural defense, not a rule
- **Category C (recurrence)**: see RECURRENCE-DETECTED above — run `/extract-lesson` with this context
- **Category D (universal) excluded** — not applicable for per-project review; belongs in global `~/.claude/CLAUDE.md` if it emerges

### Next action
No blockers → proceed to commit.
Resolve HIGH signal → re-run review-task before committing.
Lesson candidates exist → consider `/extract-lesson` after commit.
```

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Auto-fixing flagged issues | Report only. User fixes (possibly in a new AI turn). |
| Blocking the commit | Never. This is a gate, not a wall. User can commit anyway after considering findings. |
| Reviewing work in progress | Wait until task is complete. Mid-task review creates churn. |
| Running on trivial changes | Skip < 10-line changes, typo fixes, comment-only. Overhead exceeds value. |
| Running on large diffs without scoping | If diff > 500 lines, ask the user to split or explicitly confirm scope; flag-density will be overwhelming otherwise. |
| Silent approval (finding nothing, saying nothing) | Always produce a report, even if all 🟢 — the "nothing to flag" result is also useful. |
| Overlapping with `audit-repo-hygiene` | Stay scoped to the diff. Full-repo concerns belong in the periodic audit, not here. |

## Relationship to other skills

- **Follows** implementation (which may have used superpowers methodology skills)
- **Precedes** `extract-lesson` (which decides what to do with flagged candidates)
- **Complements** `superpowers:verification-before-completion` (that checks evidence of execution; this checks against project context)
- **Complements** `superpowers:requesting-code-review` (that's general code review; this is project-sediment review)
- **Distinct from** `audit-harness` (which audits harness artifacts, not diffs)
- **Distinct from** `audit-repo-hygiene` (which runs full-repo on cadence). Note: `audit-repo-hygiene` has a Phase 0 option (d) for diff-scope — but it uses its 14-dimension codebase-decay lens, whereas `review-task` uses a 4-phase project-discipline lens. For a full pre-commit gate on a non-trivial change, running both sequentially is reasonable; they don't subsume each other.

## One-sentence summary

**Every task gets a project-context review before commit — fast, focused on the diff, report-only, routed to extract-lesson when candidates surface.**
