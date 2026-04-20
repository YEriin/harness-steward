---
name: bootstrap-new-project
description: Use when starting a fresh AI-driven project from scratch, or when the user explicitly asks to "set up harness", "bootstrap", or "initialize" a project for AI coding discipline. Triggers include "new project setup", "Day 0", "starting a greenfield project with AI". Do NOT use if substantial code or `.harness/` already exists — route to adopt-into-existing-project instead.
---

# Bootstrap New Project

Walks the user through Day 0: capture architectural intent, seed `.harness/`, write a minimal `CLAUDE.md`, set expectations for human discipline that harness cannot automate.

## Core principle

**Harness is built from minimum viable state, not maximum preemptive state.** This skill establishes the smallest set of artifacts required for the macro loop to begin. Do not create harness for problems the project hasn't had yet.

## Phase 1 — Sanity check

Before any writing, confirm ALL three:

1. **Working directory** is the intended project directory (ask if unsure; don't guess).
2. **No substantial existing state** — no `.harness/`, no existing `CLAUDE.md` with real rules (an empty/boilerplate one is fine), no substantial source code beyond scaffolding.
3. **The user can articulate** (even roughly) the project's purpose in 2-3 sentences.

If #1 fails: ask the user to confirm or `cd`.
If #2 fails: **STOP**. Tell the user this project has state that needs to be respected; recommend `adopt-into-existing-project` instead.
If #3 fails: **STOP**. The Day 0 doc can't be written without this. Ask them to come back when they can name what they're building.

## Phase 2 — Architectural intent dialogue

Write architectural intent through interactive Q&A. **You ask, they answer, you transcribe.** Do not invent content — AI can help articulate once they have thoughts; AI cannot originate architectural judgment.

Cover four sections (see `references/writing-architectural-intent.md` for full guidance):

1. **What this system IS** — *"In 2-3 sentences, what is this project's central purpose?"*
2. **What this system is NOT** — *"What are 3 explicit boundaries? Each is a 'we don't do X' that prevents scope creep."*
3. **Key abstractions** — *"Name 3-5 core concepts. For each: what it does, which module owns it, and critically: what it's NOT allowed to know about."*
4. **Evolution direction** — *"Where is this heading in 3-6 months? Direction of travel, not a roadmap."*

If the user struggles with a section, offer: *"We can leave this as a `<FILL IN>` placeholder — but the document isn't load-bearing until it's filled. Would you rather pause and come back, or use a placeholder?"*

**Don't fill in for them.** The document loses all value when you write what you think they should want.

Use `templates/architectural-intent.template.md` as the skeleton.

## Phase 3 — Seed sediment

Create in the user's project (NOT in the harness-steward bundle itself):

### 3a. `.harness/architectural-intent.md`
From Phase 2 answers, using the template as skeleton. Date it today.

### 3b. `.harness/lessons-log.md`
Initialize with header and bootstrap entry:

```markdown
# Lessons Log

Chronological record of project-specific lessons. Managed by the `extract-lesson` skill.

---

## <YYYY-MM-DD> — project bootstrapped

**What:** Project bootstrapped with harness discipline.
**Why:** Establishing Day 0 sediment.
**Response:** architectural-intent.md + CLAUDE.md seeded; first macro loop scheduled.
```

### 3c. `CLAUDE.md` (project root)
From `templates/claude-md-minimal.template.md`. **Do not add project-specific rules yet** — they should earn their place via recurrence, not Day 0 speculation. Leave the `## Project-specific` section empty.

The only acceptable Day 0 exception: one "cannot forget" invariant derivable directly from the architectural intent (e.g., if intent says "we don't persist PII," a rule `"never log raw user objects"` is fair). When in doubt, skip it.

## Phase 4 — Suggest structural foundation

Detect the stack (presence of `package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, etc.). **Suggest, don't auto-create:**

- **Tests must pass** hook — before any "done" claim (language-specific command)
- **Lint must pass** hook — before committing
- **Destructive ops** confirmation — `rm -rf`, force push, DB migrations, external API writes

Give concrete example commands. Leave implementation to the user; they know their settings.json and their CI. Harness suggests; user owns.

Do NOT build the hooks automatically. That crosses the "bundle never writes structural code into user project" line.

## Phase 5 — Human-touchpoint reminders

End with explicit reminders of discipline harness cannot enforce. Pick from the list; tailor to context. **Do not skip this phase** — it's how the user remembers Layer 5 is not harness's job.

1. **Weekly macro loop** — schedule `audit-harness` + `audit-repo-hygiene` on a recurring slot (Friday PM, start of sprint, whatever fits your cadence).
2. **Solo-coding slot** — 3-5 hours per week without AI. Preserves Layer 5 judgment. See `references/developer-discipline.md`.
3. **Read every diff** — don't let AI summaries replace actually reading the code. Especially early in project when patterns are being set.
4. **After every feature** — invoke `extract-lesson` before starting the next. Prevents Layer 4 lesson evaporation.

## Exit confirmation

Before declaring the skill done, verify all of:

- [ ] `.harness/architectural-intent.md` exists; all 4 sections have content OR explicit `<FILL IN>` placeholders with a note about what's pending
- [ ] `.harness/lessons-log.md` exists with bootstrap entry
- [ ] `CLAUDE.md` exists with minimal rules (and at most one project-specific rule)
- [ ] User has been told the 4 human touchpoints from Phase 5
- [ ] Suggested hooks from Phase 4 given as concrete commands, not just mentioned abstractly

Close with: *"Day 0 done. Your next action is to start your first feature. If you have superpowers installed, `/brainstorming` is the usual entry. After the feature ships, run `/extract-lesson` before starting the next one."*

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Filling architectural intent yourself | Ask the user; use their words. You cannot originate architectural judgment. |
| Writing 30 rules into `CLAUDE.md` | Template ships 11 universal rules. Do NOT add project-specific ones on Day 0; they must earn their place via recurrence (`audit-harness` flags dilution above ~15). |
| Creating hooks without asking | Suggest, don't auto-create. User owns their `settings.json`. |
| Running this in an existing project | Phase 1 Sanity check forbids this. Route to `adopt-into-existing-project`. |
| Skipping Phase 5 (human touchpoints) | These are the only defense against Layer 5. Non-negotiable. |
| Declaring done before exit confirmation | The checklist IS the verification step. No checklist = skill didn't finish. |

## When NOT to use

- Existing code or `.harness/` → use `adopt-into-existing-project`
- User hasn't chosen a project directory or is in exploratory mode → too early
- One-off script, not a real project → harness-steward discipline is overkill for throwaway code
- User wants a "quick" setup without architectural intent — the skill insists on this step. If they won't do it, they don't need this discipline.
