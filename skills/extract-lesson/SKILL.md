---
name: extract-lesson
description: Use at the end of a feature, after resolving a bug, or when a pattern becomes clear and you want to decide how (or whether) to preserve the lesson. Triggers include "should I add a rule for this?", "how do I remember this?", "I want to make sure this doesn't happen again", "what should I document about this?".
---

# Extract Lesson

When you've just learned something worth not losing, this skill decides **how to preserve it** without contributing to rule sprawl.

## Core principle

**Most lessons should NOT become permanent rules.** The default destination for a lesson is a dated entry in `.harness/lessons-log.md`. Promotion to a permanent rule (project `CLAUDE.md`) must be earned — by recurrence, by impossibility of structural fix, and by surviving the attention-dilution test.

This skill exists because the default path ("I'll just add a rule") is what creates rule sprawl, which destroys the value of all existing rules (see `../../references/rule-memory-management.md`).

## Step 1 — Capture the lesson

Answer three questions in one paragraph. Don't proceed past this step without these.

- **What happened?** (factually, no editorializing)
- **Why?** (root cause, not symptom — "AI didn't run tests" is a symptom; "nothing structurally requires AI to run tests before claiming done" is a cause)
- **What would have prevented it?** (concrete intervention, not "be more careful")

## Step 2 — Classify the intervention

Match your "would have prevented" answer to **exactly one** of these categories:

### A. Structural — intervention can be a hook, lint rule, CI check, test, or type

Examples: "AI claims done without running tests" → PostToolUse hook; "import from deprecated module" → lint rule; "PII leaked in logs" → type or middleware.

**→ Specify the structural defense concretely — file path, command, mechanism — and propose it to the user. Do NOT auto-implement; structural code (hooks, CI, lint configs, types) lives in the project and the user owns those decisions. Do NOT write a behavioral rule as a substitute.**

A rule that duplicates structural enforcement is pure noise. If the user then says "go ahead and implement it," that's a separate request — but the skill's default output is a specification, not an edit.

### B. Architectural — the lesson refines what the system IS or IS NOT

Examples: "payment module shouldn't call billing directly" → architectural boundary; "sessions are ephemeral" → abstraction clarification.

**→ Update `.harness/architectural-intent.md`.** Log a reference in lessons-log so you know when the intent was refined.

### C. Project-specific behavioral rule — applies in this project, can't be structural

Examples: "when adding a migration, always check read-replica compatibility"; "new API endpoints need rate-limit config in Y file".

**→ Apply the attention-dilution test before adding to `CLAUDE.md`.**

Require ALL three to be true:
1. This **type** of issue has happened 2+ times. **Search `.harness/lessons-log.md` for similar root-cause `**Why:**` entries (not just similar symptoms).** If you don't find 2+, it's a first occurrence — category E, not C.
2. AI would concretely have done the wrong thing without the rule (not "would have been less elegant")
3. The rule's value is clearly greater than the attention it takes from all existing rules

If any is "no" or uncertain → treat as category E (lessons log only, await recurrence).

### D. Universal AI behavior — applies across any project you work on

Examples: "don't hallucinate API functions"; "don't claim done without verification".

**→ Belongs in global `~/.claude/CLAUDE.md` or a reference skill, NOT this project's CLAUDE.md.** Suggest the change at the right level.

### E. One-off / contextual — specific to this feature, unlikely to recur

Default category. When in doubt between C and E, choose E.

**→ Append a dated entry to `.harness/lessons-log.md`. No rule. No further action.**

## Step 3 — Execute

| Category | Action |
|----------|--------|
| A | Produce a concrete specification of the structural defense (file path, command, behavior). **Propose to user; do not auto-implement.** Log the proposal to `.harness/lessons-log.md` with `Response: proposed structural fix — <description>`. |
| B | Edit `.harness/architectural-intent.md`. Log the reference in lessons-log. |
| C (passed test) | Append to project `CLAUDE.md`. Log the promotion with reason. |
| C (failed test) | Log to `.harness/lessons-log.md`. Await recurrence. |
| D | Suggest addition to global CLAUDE.md or reference an existing skill. Don't add to project CLAUDE.md. |
| E | Log to `.harness/lessons-log.md`. |

## Lessons log entry format

Append to `.harness/lessons-log.md` (create if missing):

```markdown
## YYYY-MM-DD — brief title

**What:** one-line factual description
**Why:** root cause
**Response:** structural fix added / logged only / promoted to CLAUDE.md on YYYY-MM-DD
**Related:** (links to prior entries if this is a recurrence)
```

The **Related** field is how category C rule-promotion earns its right to exist: if you can link to 2+ prior entries on the same root cause, the rule is justified.

## Red flags — STOP before codifying

When you feel any of these, the answer is "log to lessons-log, don't codify":

- *"This is the first time, but better safe than sorry"*
- *"That was painful, I should save the pain"*
- *"Everyone should know about this"*
- *"I'll add a rule AND log it"* ← doing both = dilution without benefit
- *"This is too important not to codify"* ← urgency signal is usually wrong
- *"It's just a small rule"* ← every rule dilutes; size doesn't change that
- *"We can always delete it later"* ← you won't

**If the lesson matters, it will recur. Recurrence is what earns permanent rules. Until then, log.**

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Promoting after one incident ("lesson-panic") | Wait for recurrence. Log now. |
| Vague rule ("be more careful with X") | Either name the concrete action, or don't codify. |
| Duplicating structural enforcement | If a hook / lint / CI catches it, the rule is noise — remove. |
| Rules with no recurrence evidence | Log, let them prove value by recurring. |
| Skipping this skill ("I'll just add a rule") | The skill exists because that default creates sprawl. |
| Editing old lessons-log entries to reflect new understanding | Don't. Append a new entry that references the old one. |
| Deleting old lessons-log entries because they "never recurred" | Don't. Append-only. Quiet entries are evidence, not clutter — see next section. |

## lessons-log is append-only

`.harness/lessons-log.md` is an **append-only historical record**. Two corresponding prohibitions:

- **Don't edit** old entries to reflect new understanding. If a pattern becomes clear only later, add a new entry referencing the old one.
- **Don't delete** old entries, even if they look stale, obsolete, or never recurred. Recurrence detection — the only justification for category-C promotion — depends on being able to search history. An entry that looks quiet today is evidence that will matter tomorrow. `audit-harness` may flag dormant entries as informational; it never recommends deletion.

## When NOT to use this skill

- **Mid-feature** — lessons aren't clear until the feature is done. Wait.
- **Pure implementation details** — *"this file needs Python 3.11"* is project config, not a lesson.
- **Emotional processing** — *"that was annoying"* is not a lesson.
- **Celebrating a win** — *"that went well"* is not a lesson unless you can name what specifically to preserve.

## One-sentence summary

**Default: log to lessons-log. Promote only when you can point to recurrence, rule out structural fix, and justify the attention cost.**
