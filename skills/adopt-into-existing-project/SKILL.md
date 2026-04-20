---
name: adopt-into-existing-project
description: Use when bringing harness discipline into a project that already has code, history, and possibly existing rules/docs. Triggers include "adopt harness", "migrate to harness", "add AI coding discipline to this project", "progressively introduce harness". Do NOT use for brand-new projects — use bootstrap-new-project instead. Do NOT use if `.harness/` already exists — use audit-harness.
---

# Adopt Into Existing Project

Stages the introduction of harness discipline into a project with existing state. Unlike `bootstrap-new-project`, this respects what's already there: existing `CLAUDE.md`, existing architectural docs, existing hooks, existing conventions.

## Core principle

**Every phase produces a proposal, not an edit. User decides what to act on and when.**

Unlike `bootstrap-new-project` which sets everything up in one session, adoption into an existing project must respect what's already there. The skill runs through phases in one invocation to produce a coherent plan, but **every phase's output is a proposal that the user acts on separately** — CLAUDE.md triage is a plan (Phase 2), structural additions are suggestions (Phase 3). The user stages the actual execution; the skill stages the diagnosis.

## Phase 0 — Prerequisites and detection

Confirm:

- Working directory is the project root
- **No `.harness/` already exists** — if it does, route to `audit-harness`; the project is already harness-aware
- **User can articulate at least one pain point** (bloat, rule sprawl, inconsistency, drift, AI drifting on them) — if they can't, harness discipline is friction without perceived benefit; they may not be ready yet

Detect current state (read-only; don't write anything yet):

- Existing `CLAUDE.md` (project root) — read if present
- Existing architectural docs (`README.md` architecture section, `ARCHITECTURE.md`, `docs/`, `docs/architecture*`)
- Existing `.claude/settings.json`, hooks, local skills
- Existing pre-commit hooks (`.husky/`, `.pre-commit-config.yaml`)
- Existing lessons-log-like file (retrospectives, learnings, post-mortems)
- Project size and type (same detection as `audit-repo-hygiene` Phase 0 — but no checks yet)

**Report detection findings to the user** and confirm before writing anything. Example: *"I found an existing `CLAUDE.md` with 22 rules, `docs/architecture.md` from 2024, and a `.husky/pre-commit` hook that runs lint. I'll integrate with these, not replace them. Ready to proceed?"*

## Phase 1 — Minimum viable harness (non-disruptive writes)

**Goal: get `.harness/` sediment in place without touching existing files.**

### 1a. Architectural intent — reverse-engineering dialogue

Unlike `bootstrap-new-project`, the project already exists — you're helping the user articulate what's **already there**, not what's planned.

Ask the 4 sections (same as `../../references/writing-architectural-intent.md`), framed for existing state:

1. *"In 2-3 sentences, what IS this system today?"* — describe current reality, not aspiration
2. *"What is it explicitly NOT, based on what you've been rejecting?"* — hard boundaries you've defended
3. *"What are the 3-5 core concepts the code is actually organized around?"* — reverse-engineer from code, not from what you wish it was
4. *"Where is it heading in 3-6 months?"* — direction of travel

**If existing architectural docs are found**, read them and offer: *"Section X in your existing ARCHITECTURE.md is close to what goes here. Let me transcribe, then we refine if needed."* **Don't invent content**; use existing words where possible, prompt for missing parts.

Write to `.harness/architectural-intent.md` (use `../../templates/architectural-intent.template.md` as skeleton).

### 1b. Lessons log — initialize with adoption entry

Create `.harness/lessons-log.md`:

```markdown
# Lessons Log

Chronological record of project-specific lessons. Managed by `extract-lesson`; append-only (never edit, never delete).

---

## <YYYY-MM-DD> — harness adopted

**What:** Harness discipline introduced into existing project (N months old).
**Why:** <user's stated pain point(s) from Phase 0>
**Response:** .harness/ sediment created; existing CLAUDE.md pending review (Phase 2); first macro loop scheduled within 1-2 weeks.
```

## Phase 2 — CLAUDE.md review / consolidation (propose, don't execute)

Apply the same checks as `audit-harness` **Checks 1a-1f** (dilution, staleness, vagueness, structural redundancy, misfit, **missing context pointers**) to the existing `CLAUDE.md`:

- **1a. Count**: if total rules > 15, flag dilution risk
- **1b. Staleness**: for each rule naming a file/function/module, Grep to verify existence; flag misses
- **1c. Vagueness**: rules of the form *"be more careful"*, *"try to avoid"*, *"consider"* → flag
- **1d. Structural redundancy**: rule duplicates an existing hook / lint / CI check → flag
- **1e. Misfit**: rule references tools/frameworks not in the project → flag
- **1f. Missing context pointers**: does the existing `CLAUDE.md` reference `.harness/architectural-intent.md` and `.harness/lessons-log.md`? If not, **this is a near-certain add to the triage plan** — without the pointers, the sediment files you just created in Phase 1 will be invisible to Claude during regular project work, defeating their purpose

Unlike `audit-harness` which emits a findings report, this Phase emits a **triage plan** — concrete edit proposals organized by disposition:

```
## CLAUDE.md Triage — <N> rules scanned

### Keep as-is
- Line <X>: <rule excerpt> — <reason>

### Rewrite for clarity
- Line <X>: <rule> → suggested: <rewrite>

### Remove
- Line <X>: <rule> — <reason: stale / vague / redundant / misfit>

### Consolidate
- Lines <X> + <Y>: <overlap> → merge to: <consolidated rule>
```

**Do NOT edit `CLAUDE.md` yourself.** User reviews, user executes the edits. If the user asks for implementation help in a follow-up, that's a separate request — not within this skill's default behavior.

If **no CLAUDE.md existed**: seed from `../../templates/claude-md-minimal.template.md` (same as `bootstrap-new-project` Phase 3c). Do not add project-specific rules yet — let them earn their place via recurrence.

## Phase 3 — Structural foundation suggestions (propose, don't create)

Detect stack. Detect existing hooks, CI, lint configs.

Propose additions for gaps:

- **Tests-must-pass hook** — if not already enforced via hook/CI
- **Lint-must-pass hook** — if not already enforced
- **Destructive-op confirmation** — if no equivalent guard exists for `rm -rf`, force push, DB migrations, external API writes

Give concrete commands. Note explicitly: *"if a suggested mechanism already exists in your project, skip it — this is not replacement for what works."*

**Do NOT implement.** Suggestions only.

## Phase 4 — Human touchpoint reminders

Same as `bootstrap-new-project` Phase 5, plus adoption-specific:

1. **Schedule first macro loop** — run `audit-harness` and `audit-repo-hygiene` within 1-2 weeks. **Expect a long list of findings**; this is normal for newly-adopted projects.
2. **Solo-coding slot** — 3-5 hours/week without AI. See `../../references/developer-discipline.md`.
3. **Read every diff**.
4. **Invoke `extract-lesson`** after every feature — especially in the first month, when adoption-specific lessons surface rapidly.

**Adoption-specific warning**: *"Expect a burst of lesson-generation in weeks 1-4. Most insights will feel important in the moment. Pipe them through `extract-lesson` — most belong in lessons-log, not `CLAUDE.md`. If you hoard rules in weeks 1-4, you've already failed. The `attention-dilution` principle applies from day one of adoption."*

## Exit confirmation

Before declaring the skill done, verify:

- [ ] `.harness/architectural-intent.md` exists with 4 sections (FILL IN placeholders acceptable if noted)
- [ ] `.harness/lessons-log.md` exists with adoption entry including user's stated pain point
- [ ] Existing `CLAUDE.md` review plan produced (Phase 2) OR minimal CLAUDE.md seeded if none existed
- [ ] Structural suggestions given as concrete commands (Phase 3)
- [ ] 4 human touchpoints communicated (Phase 4)
- [ ] Adoption-specific warning given about week 1-4 lesson burst

Close with: *"Adoption Phases 0-4 complete. Your action items: review the Phase 2 triage plan, evaluate the Phase 3 structural suggestions, and schedule the first macro loop in 1-2 weeks (`audit-harness` + `audit-repo-hygiene`). Expect iteration — the first audit will reveal more than you expect, and that's the feature."*

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Overwriting existing `CLAUDE.md` | NEVER. Phase 2 produces a triage PLAN. User executes edits. |
| Silently running structural edits while producing the plan | Every phase is proposal-only. User executes between phases (or all at once after Phase 4); skill never writes project code. |
| Inventing architectural intent | Read existing docs; transcribe user's words. No origination. |
| Duplicating existing hooks in Phase 3 suggestions | Detect existing; don't propose what works already. |
| Flooding user with 30 CLAUDE.md changes in Phase 2 | Prioritize: HIGH (stale + misfit) → MEDIUM (vague + redundant) → LOW (consolidation). Over-prescription causes abandonment. |
| Skipping the week 1-4 lesson-burst warning | This is the expectation-set that prevents rule-panic. Non-negotiable. |
| Pre-filling architectural intent from code analysis without user input | You can suggest text pulled from existing docs — you cannot originate. If the user can't answer a section, use `<FILL IN>` marker. |

## When NOT to use

- **Brand-new project** — use `bootstrap-new-project`
- **`.harness/` already exists** — use `audit-harness`
- **One-off script / throwaway code** — harness discipline is overkill
- **User can't articulate a pain point** — harness is friction without perceived benefit; they'll abandon it within weeks. Offer to come back when they can name what hurts.
- **Project is about to be retired** — don't adopt something you're about to delete

## Relationship to other skills

- **Same conventions as `bootstrap-new-project`** for sediment creation (architectural intent, lessons-log, CLAUDE.md) and human touchpoints
- **Borrows `audit-harness` Checks 1a-1f** for Phase 2 CLAUDE.md review — same checks, different output (triage plan vs. findings report)
- **References `extract-lesson`** for post-adoption lesson handling (especially the week 1-4 burst)
- **Defers `audit-repo-hygiene`** to after Phase 4 (in 1-2 weeks) — running it inside this skill would overwhelm the user with findings before the new sediment has stabilized
