# Changelog

All notable changes to the `harness-steward` bundle will be documented here. This project follows [semver](https://semver.org/).

## [0.1.0] - 2026-04-20

Initial beta release. Nine skills across four categories: setup, workflow (ambient activation at common dev moments — evaluate-requirement, scope-implementation, review-task, debug-with-history), periodic macro audit, and lesson capture. Not yet validated in real-world projects — treat as beta until that happens.

### Added

**Setup (one-time):**
- **Bootstrap skill** `bootstrap-new-project` — Day-0 setup for new AI-driven projects; writes `.harness/architectural-intent.md`, `.harness/lessons-log.md`, minimal project `CLAUDE.md`; suggests structural hooks without auto-creating them.
- **Adoption skill** `adopt-into-existing-project` — staged introduction of harness-steward discipline into existing codebases; proposal-only for `CLAUDE.md` triage and structural suggestions; includes adoption-specific week 1-4 lesson-burst warning.

**Workflow (activates at common dev moments):**
- **`evaluate-requirement`** — PRD / new requirement evaluation against architectural intent + historical precedent; produces GO / MODIFY / DEFER / REJECT decision support. Runs BEFORE implementation planning.
- **`scope-implementation`** — pre-implementation briefing: loads architectural boundaries, past lessons, and existing conventions into conversation before coding starts. Hands off to superpowers:writing-plans or TDD.
- **`review-task`** — per-task project-aware review after implementation completes, before commit. Scans the diff against project sediment; surfaces INTENT-VIOLATION, DRIVE-BY, NO-EVIDENCE, POSSIBLE-BANDAID, RECURRENCE-DETECTED, etc. Report-only.
- **`debug-with-history`** — project-aware debug entry point. Frames the bug, searches lessons-log for similar root causes, checks architectural-intent for boundary-manifestation signals. Hands off to superpowers:systematic-debugging with context pre-loaded.

**Periodic macro:**
- **Audit skills** `audit-harness` and `audit-repo-hygiene` — periodic health checks; report-only. `audit-harness` scans rules/intent/lessons-log/settings (Checks 1a-1f including MISSING-CONTEXT-POINTERS); `audit-repo-hygiene` runs 14-dimension codebase decay scan (P0/P1/P2).

**Lesson capture:**
- **`extract-lesson`** — decision tree for preserving lessons without rule sprawl; default destination is append-only `.harness/lessons-log.md`; promotion to `CLAUDE.md` gated by attention-dilution test and structural-defense preference.
- **8 reference documents**:
  - `six-layer-failure-model.md` — AI coding failure taxonomy (generation / intent / verification / entropy / human atrophy / workflow)
  - `three-loop-workflow.md` — macro / meso / micro development loops
  - `relationship-to-superpowers.md` — scope division between harness-steward and superpowers
  - `developer-discipline.md` — role shift, iron rules, crash scenarios
  - `harness-principles.md` — structural vs behavioral, MVH, blast-radius intensity, ambient-discovery rule
  - `rule-memory-management.md` — attention-dilution principle, activation vs resident, GC cadence
  - `writing-architectural-intent.md` — Day-0 intent doc structure and maintenance
  - `finding-tag-glossary.md` — centralized reference for all structured finding tags emitted across skills
- **2 templates** — architectural-intent skeleton with `<FILL IN>` markers; minimal `CLAUDE.md` with 11 universal rules across 6 sections covering epistemic honesty, computational reliability, implementation discipline (read-before-edit / evidence-with-claim / edge-case enumeration / root-cause fixing / no drive-by), destructive-operations gate, architecture ownership, and rule meta — plus a Project Context pointer to `.harness/` sediment.
- **1 worked example** — `examples/minimal-web-service/` showing what `.harness/` sediment looks like in a live project.

### Locked design decisions (enforced across all skills)

- Universal vs sediment boundary — bundle never writes to itself; project specifics live in `.harness/`
- Structural code (hooks, CI, lint, types) is proposed/specified by skills, never auto-implemented
- `lessons-log.md` is append-only — never edit, never delete historical entries
- Audits are report-only; never modify project files

### Known limitations

- Not yet validated in real-world projects beyond subagent-level TDD; treat as beta
- Only one worked example (Node.js web service); no Python / Go / Rust / doc-only examples yet
- No `audit-this-bundle` meta-skill yet (planned for v0.2)

### Development process

- Every skill developed with per-skill TDD (baseline vs skill-loaded subagent scenarios)
- Cross-skill consistency pass introduced after v1 round-3 surfaced 3 inter-skill contradictions; mandatory from skill 4 onward
