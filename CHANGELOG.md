# Changelog

All notable changes to the `harness-steward` bundle will be documented here. This project follows [semver](https://semver.org/).

## [0.2.0] - 2026-04-21

### Changed

First real-world feedback pass on `audit-repo-hygiene` (Orbito, 1,138-file mixed Go+TS+Markdown repo) surfaced four verifiable error classes across one production run:

1. **Identifier fabrication on correct anchors** — `path:line` accurate, named symbol invented (e.g., `SeedCRM` reported at a line where the real function was `seedSampleData`; "14 `ValidationError` types" where the true count was 6)
2. **Severity inflation via charged adjectives** — `retired`, `broken`, `highest-impact` attached without evidence
3. **Scope-blind false positives** — hard-coded exclusions missed `.gitignore`'d paths and non-standard source roots (e.g., deps flagged "unused" because the scan never looked at `<project>/ux-eval/`)
4. **Recall gap on P0 config literals** — token sweeps stopped at the first scoped match; drift in CHANGELOG / CONTRIBUTING / doc trees went undetected

Each mapped to one of two structural gaps — no post-scan verification pass, scope-is-author-time — and is now closed by:

- **New reference** `references/scan-output-schema.md` — contract for scan-only dispatched agents. Line format `[TAG] path:line — descriptor — identifier:<name?> evidence:<regex|cmd|url>`, plus aggregator-side verification steps V1 (identifier grep-verification at `line ± 5`), V2 (prescriptive-claim probes: `git check-ignore`, `test -e`, cross-source-root dep grep), V3 (severity-language constraint — charged adjectives require evidence or get neutralized; severity is set by the aggregator, never by the sub-agent), V4 (unfiltered whole-repo sweep, **bounded to P0 literal-referencing findings**). V2 probe table is pattern-first with illustrative examples; language-agnostic extension guidance included. Manifest detection is ecosystem-agnostic — Java/Maven, Gradle, Ruby, PHP, Elixir, Julia, Deno, Bun, Haskell, OCaml, Crystal, .NET all appear as examples, list is explicitly non-exhaustive.
- **`audit-repo-hygiene`** — Phase 0 now reads `.gitignore` + `.git/info/exclude` and detects source roots dynamically from every directory carrying an ecosystem manifest (honors `pnpm-workspace.yaml`, `go.work`, Cargo workspace, `lerna.json`, `nx.json`, `turbo.json`, Gradle `settings.gradle*`, Maven reactor). New Phase 4 — Verification pass (V1–V4) executes before any finding is emitted. Output format carries `identifier:` + `evidence:` fields and a Dropped-categories summary so verification catches are visible. Phase 4 body kept compact in the skill (bullets: pattern + why); full tables live in the schema reference to avoid duplication.
- **`audit-harness`** — Check 1b (rule staleness) and Checks 4a/4b (dead-hook, stale-permission) now carry self-contained verification rules with scoped greps that avoid two symmetric failure modes:
  - **Self-match**: `RULE-STALE` excludes rule-source files (`CLAUDE.md`, nested `*/CLAUDE.md`, `AGENTS.md`, `.claude/` rule includes); `STALE-PERMISSION` excludes `.claude/settings.json` and `.claude/settings.local.json`. Without these, the rule / declaration self-matches and the finding cannot fire.
  - **False-positive-evergreening via historical text**: both greps exclude docs (`*.md`, `*.rst`, `*.txt`), CHANGELOG / HISTORY, and `.harness/lessons-log.md` — prose mentions don't prove the identifier is still doing work. `STALE-PERMISSION` additionally scopes the grep to the execution surface (hooks, source roots, build manifests and their script entries, shell/ops scripts, CI/deploy configs) rather than "anywhere in the repo."
  - Tag renamed `STALE-REFERENCE` → `RULE-STALE` for consistency with glossary and output example.
- **`review-task`** — Phase 0 now records `review_diff_range` (`HEAD` for uncommitted work, `HEAD~1..HEAD` or the user-specified range for committed work). INTENT-VIOLATION (Phase 1a) and DRIVE-BY (Phase 2a) findings require grep-verification before emit: named imports must match the actual text at the claimed `path:line`; drive-by files must actually appear in `git diff --name-only $review_diff_range`. Using the recorded range (rather than hardcoded `HEAD`) prevents false-verification when reviewing committed work with unrelated dirty edits in the working tree.
- **`references/finding-tag-glossary.md`** — documents the new `scope-incomplete` annotation emitted by Phase 4 V4.

### Notes

User-visible changes for skill consumers:
- `audit-repo-hygiene` output now carries `identifier:` + `evidence:` fields and a Dropped-categories block. Any downstream parser that treated findings as free text will ignore the new fields; parsers that want to use the structure should read `references/scan-output-schema.md`.
- `scope-incomplete` is a new annotation (not a tag) that may appear on P0 findings.
- `STALE-REFERENCE` was renamed to `RULE-STALE` — the old name was only introduced in this same change set and had no prior emitters, so no migration is needed.

`.harness/` sediment format, CLAUDE.md rules, and the report-only policy are unchanged. Real-world validation credit: the reporter surfaced every class with reproducible `grep` evidence, which is what made structural rather than behavioral fixes tractable.

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
- No `audit-this-bundle` meta-skill yet (now targeted for v0.3; v0.2 prioritized the field-feedback fix set)

### Development process

- Every skill developed with per-skill TDD (baseline vs skill-loaded subagent scenarios)
- Cross-skill consistency pass introduced after v1 round-3 surfaced 3 inter-skill contradictions; mandatory from skill 4 onward
