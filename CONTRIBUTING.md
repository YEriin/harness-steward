# Contributing to harness-steward

Thanks for considering a contribution. This bundle is early-stage (v0.2.0 beta); the highest-value contributions right now are **real-world validation reports** and **skill bug reports** from actual projects using it. The 2026-04-21 Orbito report is the model — reproducible `grep` evidence let the fix be structural (see `CHANGELOG.md [0.2.0]` and `references/scan-output-schema.md`).

## How it was built

The bundle was developed over ~20 commits using its own locked design decisions:

- **Universal vs sediment** — the bundle never writes to itself; all project-specific content lives under `.harness/` in user projects
- **Structural over behavioral** — skills propose / enforce structure; rules in CLAUDE.md are fallback discipline when structure isn't possible
- **`lessons-log.md` is append-only** — never edit, never delete historical entries
- **Audits are report-only** — never modify project files
- **Stack-agnostic** — every skill works across language / framework

See `references/harness-principles.md` for the full philosophy.

## Per-skill development process

Every skill in this bundle was produced using this four-step loop:

1. **Draft** — write `SKILL.md` content
2. **Consistency pass** — dispatch a subagent to read the bundle's existing skills + the new draft, checking for:
   - Conflicting directives (e.g., one skill saying "build X", another saying "don't build X")
   - Terminology drift (same concept with different names across skills)
   - Broken cross-references (references to skills / files / conventions that don't exist)
   - Overlapping scope without clear division
   Act on findings before TDD.
3. **TDD validation** — dispatch paired subagents (baseline without skill / skill-loaded) on a representative scenario; verify the skill causes meaningful behavior change vs. baseline.
4. **Commit** with both consistency-pass and TDD results in the message.

Please use the same process if you're adding / changing a skill. The consistency pass catches cross-skill inconsistencies that per-skill TDD cannot detect.

## Real-world validation reports

If you've run this bundle on a real project (even just one feature end-to-end), **please open an issue** with:

- What project type (Web service / library / CLI / etc. — stack-agnostic means we want diverse examples)
- Which skills triggered naturally vs. had to be invoked explicitly
- Where a skill's output felt right vs. wrong
- Any friction you noticed (descriptions that mismatched the moment, false negatives, duplicated work across skills)

This is the gate between the current v0.x beta line and v1.0 (stable). Subagent TDD catches obvious wrongness; only real use catches the subtle kind — v0.2.0 proved the point.

## Skill bug reports

If a skill fired at the wrong moment, produced wrong findings, or referenced files that don't exist in your project, open an issue with:

- Skill name
- User input that triggered (or should have triggered) it
- What happened vs. what you expected

## Adding a new skill

Before adding, run through:

1. Does it fit the existing category map (Setup / Workflow / Periodic / Lesson capture)?
2. Could an existing skill absorb it?
3. Does it duplicate superpowers' scope? If so, scope it narrower (harness is project-aware; superpowers is methodology-only)
4. Will it earn its place? One way to check: has an equivalent of this skill already been used ad-hoc 2+ times in real projects?

If yes to all, draft using the per-skill process above.

## Adding a new reference or template

Same bar as adding a skill — does it earn its place? Reference docs have lower signal-to-noise but still consume attention. Before adding:

- Is there existing reference content that covers this?
- Can it be a section in an existing reference instead of a new file?

## Dogfooding

This bundle uses its own discipline on itself. If you're working on bundle changes:

- Run `audit-repo-hygiene` (or apply its logic manually) before big changes
- Run `review-task` post-commit to catch drive-bys, missing evidence, etc.
- When debugging an issue in the bundle, use `debug-with-history` and search this repo's git history + CHANGELOG

We'll add a formal `.harness/` sediment directory to this repo in a future release so bundle development can go through the full bundle workflow.

## Versioning

Semver. The bundle is at 0.2.0 (beta). 0.1.0 was the initial skill bundle; 0.2.0 folded in the first field-feedback hardening pass (four verifiable error classes closed, scan-output-schema contract added). 1.0 signifies "stable enough to recommend broadly" and is gated on additional real-world validation across varied stacks.
