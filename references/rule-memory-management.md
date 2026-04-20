# Rule and Memory Management

A central claim of this bundle: **rule quality matters more than rule quantity.** This document captures the principles for keeping rules (CLAUDE.md entries, memory files, skill docs) from slowly suffocating themselves.

## The attention-dilution principle

**Every rule added to an active context dilutes attention from all existing rules.**

This is the reason rule-sprawl is dangerous even when every individual rule seems sensible. A rule isn't "free" just because it's short — the cost is paid by every *other* rule losing a fraction of attention share.

Before adding a rule, the bar is:

> **Is this rule's value greater than the attention it dilutes from all existing rules combined?**

If the answer isn't clearly yes, don't add it. Write it to `.harness/lessons-log.md` instead (see `extract-lesson`), and promote it later only if the same issue actually recurs.

## Activation-based vs resident rules

Rules can live in two forms:

| Form | Loaded | Use for |
|------|--------|---------|
| **Resident** | Every session | Foundational invariants — 11 universal rules from the minimal template; project-specific additions earned by recurrence, total capped ~15 per `audit-harness` dilution threshold |
| **Activation-based** | When relevant (via skill, hook, or description match) | Domain-specific discipline, workflow patterns |

Most failure comes from putting everything resident. `CLAUDE.md` becomes a 300-line wall of rules, and model attention gets sliced so thin that load-bearing ones get ignored anyway.

**Default choice: activation-based.** Only promote to resident if the rule applies *literally every session*.

## Memory layering

Resident context lives in multiple layers:

- **Global** (`~/.claude/CLAUDE.md`) — user-identity rules (*"I prefer X language"*, *"never force-push"*)
- **Project** (`<project>/CLAUDE.md`) — project-specific invariants
- **Session** (in-conversation) — ephemeral

Higher layers override lower. **Conflicts between layers should be rare and explicit** — if you find yourself wanting a project rule that contradicts a global rule, something deeper is off.

## Rule GC cadence

Rules rot. Specifically:

- Rules reference removed or renamed functions
- Rules capture a principle that's now enforced structurally (via hook / lint) — redundant
- Rules apply to a concern the project no longer has
- Rules were added cargo-culted from elsewhere and never fit

**GC cadence: every macro loop** (weekly or monthly). `audit-harness` is the skill that does this. For each rule, it asks:

1. Does the thing it references still exist?
2. Is this now enforced structurally elsewhere?
3. Has the project actually been bitten by this rule's absence in the last N weeks?

If "no" to any of these, the rule is a candidate for removal.

## Common failure modes

- **Hoarding lessons** — every meeting produces 3 new rules; `CLAUDE.md` balloons to 300 lines in 3 months
- **Asymmetric accumulation** — rules get added but never removed; only one half of the hygiene cycle runs
- **Stale references** — rules mention `OldService.parse()` that was renamed 6 weeks ago; AI now reasons from a fictional API
- **Cargo-cult imports** — rules copied from a blog post or another team, never fitted to this project
- **Redundant structural enforcement** — a rule says *"don't X"*, but a pre-commit hook already prevents X; the rule is noise
- **Lesson-panic** — adding a permanent rule after a single painful incident that's unlikely to recur

## The "don't codify everything" default

Most lessons worth remembering are **not worth codifying as permanent rules.** `extract-lesson` encodes the decision tree explicitly.

**Default behavior:**

1. Write the lesson to `.harness/lessons-log.md` as a dated note
2. Promote to `CLAUDE.md` **only after the same issue actually recurs** (not after the first time it hurts)
3. Sunset the `CLAUDE.md` rule later if the structural fix eventually makes it redundant

This default protects attention from being diluted by one-off learnings that feel profound in the moment but won't apply again.

## Relationship to bundle skills

- `bootstrap-new-project` — seeds `CLAUDE.md` with the 11-rule universal template; no project-specific rules on Day 0
- `audit-harness` — runs periodic GC and flags attention-dilution risk
- `extract-lesson` — gates the "promote to CLAUDE.md" decision

## One-sentence summary

**Rule management is signal-to-noise engineering: every new rule must pay for the attention it takes from all existing rules, and old rules must be actively pruned or they poison the ones you need.**
