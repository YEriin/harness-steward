---
name: audit-harness
description: Use periodically (weekly or monthly) or when the user says the CLAUDE.md / rules / harness config feels bloated, stale, or ignored. Triggers include "audit rules", "CLAUDE.md is noisy", "harness review", "rule GC", "is my harness still useful". Runs a structured check across rules, architectural intent, lessons log, and settings; reports findings but never auto-fixes.
---

# Audit Harness

Periodic health check for your project's **harness artifacts** — rules, memory, hooks, settings, architectural intent, lessons log. Run weekly for active projects, monthly for stable ones.

## What this audits vs. audit-repo-hygiene

- **This skill** — your harness itself: `CLAUDE.md`, `.harness/*`, `settings.json`, project-local skills
- **`audit-repo-hygiene`** — the codebase: code, docs, tests, dependencies (14 dimensions)

Run both for a full macro loop. If time-constrained, run this one first — harness health shapes everything else.

## Scope detection

Before any checks, confirm:
- Project root identified (where `CLAUDE.md` lives)
- `.harness/` directory exists — if missing, **STOP**. Route to `bootstrap-new-project` or `adopt-into-existing-project`.
- `.claude/settings.json` and any local skills noted for later checks
- Global user-level config (`~/.claude/`) is NOT in scope; audit separately

## Check 1 — CLAUDE.md rule hygiene

Read project `CLAUDE.md`. Apply in order:

### 1a. Attention-dilution risk
Count total rules. The minimal-template baseline is **11 universal rules across 6 sections** (plus a Project Context pointer section); rules beyond that baseline are project-specific additions that should be earned through `extract-lesson` recurrence, not speculated on Day 0.
- `> 15 rules` — ATTENTION-DILUTION risk (≥ 4 project-specific additions beyond baseline); investigate individual rules below
- `> 25 rules` — HIGH DILUTION, GC strongly recommended before other work

### 1b. Staleness
For each rule that names a file, function, module, config key:
- Use Grep/Glob to verify the referenced identifier exists
- Not found → flag **STALE-REFERENCE**

### 1c. Vagueness
Rules of the form *"be more careful with X"*, *"try to avoid Y"*, *"consider Z"*, *"prefer A over B when appropriate"* → flag **VAGUE** (no concrete action = no testable compliance = worthless rule).

### 1d. Structural redundancy
For each behavioral rule, check if a hook, lint config, type system, or CI already enforces it:
- If yes → flag **REDUNDANT** (the rule is noise; the structural defense is truth)

### 1e. Misfit / cargo-cult
Rules mentioning tools/frameworks/patterns the project doesn't use (inspect `package.json`, imports, file types) → flag **MISFIT**.

### 1f. Project Context pointers
Does `CLAUDE.md` reference `.harness/architectural-intent.md` and `.harness/lessons-log.md`? If either is missing → flag **MISSING-CONTEXT-POINTERS**. Without these pointers, Claude does not discover the sediment files during regular project work (only when a harness-steward skill is explicitly invoked), which defeats the purpose of maintaining them.

Remediation: propose adding the **Project Context** section from `../../templates/claude-md-minimal.template.md` to the top of `CLAUDE.md`.

## Check 2 — architectural-intent drift

Read `.harness/architectural-intent.md`:

### 2a. Fill completeness
Any `<FILL IN>` markers remaining? → flag **INTENT-INCOMPLETE** (doc isn't load-bearing until filled).

### 2b. Freshness
Check `Last reviewed` date vs. `git log` since that date:
- > 90 days AND ≥ 30 commits since last review → flag **INTENT-STALE**

### 2c. Module-ownership violations
For each *"Lives in X; does not know about Y"* claim in Key Abstractions:
1. Glob for the X module; confirm it exists
2. Grep from X-module files for imports of Y
3. If imports found → flag **INTENT-VIOLATION** (boundary breach)

## Check 3 — lessons-log patterns

Read `.harness/lessons-log.md`. **`lessons-log.md` is append-only by design.** This audit flags patterns for attention; it never recommends deletion of historical entries (see `extract-lesson` → "lessons-log is append-only").

### 3a. Recurring unaddressed root cause
Group entries by similar `**Why:**` root cause. If 2+ entries share one AND all have `**Response:** logged only` → flag **PROMOTION-CANDIDATE** (per `extract-lesson` category A or C criteria, this has earned a structural fix or project rule).

### 3b. Dormant entries (informational only)
Entries older than 6 months with `logged only` response and no later related entries → flag **LESSON-DORMANT**.

Dormant is a **signal about historical quietness**, not a **deletion recommendation**. Old entries retain value for recurrence detection even when quiet; removing them would undermine the category-C promotion path, which relies on searching history for repeat root causes. If the user explicitly asks to archive or prune, they do so manually — the audit does not propose it.

## Check 4 — settings and hooks

Read `.claude/settings.json` (or equivalent):

### 4a. Dead hook references
Hooks referencing scripts/commands that don't exist on disk / aren't on PATH → flag **DEAD-HOOK**.

### 4b. Stale permissions
Permission rules granting access to tools, paths, or commands the project no longer uses → flag **STALE-PERMISSION**.

### 4c. Local skills
For any project-local skills defined in `.claude/skills/` or similar: verify their referenced files/paths still exist. Don't audit their logic — that's out of scope.

## Output format

```
## Harness Audit — <project name>
**Date:** <YYYY-MM-DD>
**Scanned:** CLAUDE.md (<N> rules), .harness/*, .claude/settings.json

### 🔴 High signal (act first)
- [MISSING-CONTEXT-POINTERS] CLAUDE.md has no reference to .harness/architectural-intent.md or .harness/lessons-log.md — sediment is invisible to Claude during regular work
- [RULE-STALE] CLAUDE.md:12 — references `OldService.parse()`, not found in codebase
- [RULE-REDUNDANT] CLAUDE.md:19 — "always run tests"; hook `.claude/hooks/test-gate.sh` already enforces this
- [INTENT-VIOLATION] jobs/scheduler.ts imports from http/ — intent says Job does not know about HTTP
- [PROMOTION-CANDIDATE] lessons-log: 3 entries on "backend/frontend type mismatch" — propose structural fix (shared schema package)

### 🟡 Medium signal (review)
- [RULE-VAGUE] CLAUDE.md:24 — "be more careful with async" → specify concrete action or remove
- [INTENT-STALE] architectural-intent.md last reviewed 2025-08; 47 commits since

### 🟢 Low signal (informational)
- [LESSON-DORMANT] lessons-log 2025-10-03 never recurred (historical record; kept by design)
- [STALE-PERMISSION] settings.json grants `legacy-db` commands; no longer used

### Summary
| Category | Count | Threshold |
|----------|-------|-----------|
| Rules | 14 | <10 ideal, <20 acceptable |
| Intent fill-ins | 0 | 0 |
| Dead hooks | 0 | 0 |
| Promotion candidates | 1 | — |
```

## Auto-fix policy

**This skill does NOT auto-fix. It reports; the user decides.**

Reasons:
1. Harness changes have second-order effects (removing a rule may unblock the drift it was masking; removing a permission may break an unrelated workflow)
2. Architectural judgment is human work (Layer 5 defense)
3. Promotion decisions from lessons-log belong to `extract-lesson`'s decision tree, not to this auditor

The single narrow exception: `<FILL IN>` markers are reported but **never auto-filled by AI** — filling them would originate architecture.

## Red flags — stop if you catch yourself doing these

- Editing CLAUDE.md directly while running the audit — don't. Separate concerns.
- "This rule is obviously wrong, let me just remove it" — report it, let the user decide
- Promoting "medium" findings to "high" because they feel alarming — resist. Under-report is better than over-report.
- Adding new rules as part of the audit — out of scope. Rule addition is an `extract-lesson` decision.

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Running daily | Weekly / monthly. Daily is noise. |
| Auto-removing flagged rules | Report only. User decides. |
| Skipping when "everything feels fine" | That feeling is often the drift itself. Run anyway. |
| Editing or deleting lessons-log entries to reflect new understanding | Don't. `lessons-log.md` is append-only; add new entries, never modify or remove old ones. |
| Missing `.harness/` → invent one | Route user to bootstrap / adopt instead. |

## Exit

End with: *"Audit complete. Act on 🔴 HIGH signal findings first. Schedule next run: [date 1-4 weeks out, based on project velocity]."*

If promotion candidates were flagged, suggest: *"Category-A or C promotion candidates found. Next action: invoke `/extract-lesson` on the recurring pattern(s) to decide structural fix or rule promotion."*

## When NOT to use

- No `.harness/` — use `bootstrap-new-project` or `adopt-into-existing-project`
- Mid-feature crunch — audits need attention-bandwidth you don't have
- Within 1 week of a clean audit — too soon; false noise
