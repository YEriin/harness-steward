---
name: audit-repo-hygiene
description: Use periodically (weekly/monthly) to detect codebase decay — broken cross-references, dead code, duplication, drift between docs and reality, dependency bloat, test quality, error-handling inconsistency. Triggers include "audit code", "repo hygiene", "find rot", "is the codebase decaying". Report-only; never auto-fixes.
---

# Audit Repo Hygiene

Periodic scan for codebase decay across 14 dimensions. Complements `audit-harness` (which scans harness artifacts). Run both for a full macro loop. If only time for one this week: run `audit-harness` first — harness health shapes everything else.

## Scope

This skill audits **code and docs** (codebase entropy — Layer 4):
- Cross-reference integrity, consistency, hidden dependencies, config sprawl
- Dead code/content, doc-reality drift, duplication, dependency bloat, abstraction pressure
- Module boundaries, hierarchy bloat, test superficiality, error-handling consistency, comment rot

Does NOT audit:
- Rules, memory, hooks, settings → `audit-harness`
- Runtime behavior, production incidents → not harness's job
- Code style / formatting → existing linters

## Policy — report-only

**This skill reports findings. It does NOT auto-fix.** Even for mechanically-safe fixes (broken link with 1 rename candidate, obvious dead code, etc.).

Reasons:
1. Every "safe" fix has second-order effects; user should see the decision
2. Auto-fix collapses the review pass that keeps you sharp (Layer 5)
3. Bundle-wide consistency: no harness-steward skill writes structural code into user projects

If the user wants fixes applied, they start a separate session after reading the report. The two are kept separate on purpose.

## Prerequisites

- `.harness/` exists (if not → route to `bootstrap-new-project` or `adopt-into-existing-project`)
- Working directory is the project root

## Phase 0 — Project scan

Detect:
- **Project type**: doc-only (>80% markdown/txt/rst, no manifest) / code-only (<10% doc, has manifest) / mixed
- **Primary language(s)** from file extensions
- **Key files**: manifests (`package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `Cargo.toml`), config (`.env*`, `*.yaml`, `*.toml`), tests (`test*/`, `*_test.*`, `*.test.*`, `*.spec.*`)

**Scope gate** — if `>500 files` (excluding `node_modules`, `.venv`, `vendor`, `dist`, `build`, `.git`) and no scope arg was given, ask the user:
- (a) whole repo
- (b) most recently modified 200 files (via `git log`) plus all doc files
- (c) specific subdirectory
- (d) only files in current diff (`git diff --name-only HEAD`)

If args were passed (e.g. `diff`, `src/api`, `only P0`), interpret as scope/filter hints without asking.

Skip code-side checks for doc-only repos; skip doc-side checks for code-only repos.

## Phase 1 — Critical checks (P0)

### C1. Cross-reference integrity [both]
- Doc: markdown links + image refs → verify target file exists; if anchor, verify heading exists
- Code: local imports → verify module exists
- Flag: **BROKEN-REF** | If exactly 1 same-basename + shared-path-segment candidate exists, note as rename candidate (still report-only)

### C2. Consistency contradictions [both]
- Doc: same term/rule defined in 2+ files with different values
- Code: same constant/env-var in 2+ places with different values
- Flag: **CONFLICT**

### C3. Implicit dependencies [both]
- Doc: file A uses a term defined (as heading) in file B without cross-reference
- Code: global/singleton usage without explicit import
- Flag: **IMPLICIT-DEP**

### C4. Config sprawl [code]
- Same logical config key appears in 2+ sources (`.env*`, config files, hardcoded constants, `process.env.*` / `os.environ`)
- Flag: **CONFIG-SPRAWL** with all locations

## Phase 2 — Quality checks (P1)

### C5. Dead content / dead code [both]
- Doc: TODO/FIXME/HACK past dated deadlines or for features that exist; commented-out blocks > 5 lines
- Code: exported functions/classes with 0 references — **respect do-not-touch set** (package.json `exports`/`main`/`bin`, framework convention paths like Next.js `app/**/page.*`, Remix `app/routes/**/*`, SvelteKit `src/routes/**/+*`, Astro `src/pages/**/*`, Nuxt `{pages,layouts,middleware,plugins}/**/*`, `*.d.ts`, `*.config.*`, `**/{plugins,hooks,middleware}/**`)
- Flag: **DEAD-CODE** or **DEAD-CONTENT**

### C6. Doc-reality drift [both]
- Directory tree listings in docs vs. actual filesystem
- JSDoc/docstring `@param` names vs. actual signatures
- Feature lists vs. code
- Flag: **DRIFT**

### C7. Content / logic duplication [both]
- 5+ consecutive identical lines in 2+ files
- Rules sharing >60% key terms in different docs
- Function pairs with 4+ char shared prefix and length diff ≤ 2 (e.g., `parseDate`/`parseDates`)
- Flag: **DUPLICATION**

### C8. Dependency bloat [code]
- Dependencies with 0 imports in source (excluding manifest, lockfiles, `node_modules`/`.venv`/`vendor`) → **UNUSED-DEP**
- Known overlap pairs (`axios`+`node-fetch`+`got`, `moment`+`dayjs`+`date-fns`, `lodash`+`underscore`, `requests`+`httpx`, etc.) → **OVERLAP-DEP**

### C9. Abstraction pressure [code]
- Utility/helper functions with exactly 1 call site → **OVER-ABSTRACTION**
- Identical 5+ line blocks in 3+ locations (reuse C7 findings) → **UNDER-ABSTRACTION**

## Phase 3 — Maintainability checks (P2)

### C10. Module boundary overlap [both]
- Doc: 3+ files substantially covering the same topic
- Code: same concern (auth, validation, logging, rate-limit, cache, etc.) spread across 3+ files without a clear authoritative home
- Flag: **BOUNDARY-DIFFUSE**

### C11. Hierarchy bloat [both]
- Doc: files > 500 lines, heading depth ≥ 5, directories > 3 levels with < 20 files
- Code: files > 400 lines, functions > 50 lines, max indentation > 5
- Flag: **BLOAT**

### C12. Test superficiality [code]
- Test files where mock/stub setup lines / total assertion lines > 70%
- Test files averaging < 2 assertions per test
- Tests that don't reference the module they claim to test
- Flag: **TEST-SHALLOW**

### C13. Error-handling inconsistency [code]
- Catalog patterns (try/catch, `.catch()`, return-error-object, silent log-only catch, `except: pass`)
- Flag when 3+ patterns coexist in similar call paths → **ERROR-INCONSISTENT**
- Empty catch / pass-only catch → **SILENT-FAIL**

### C14. Comment rot [both]
- HTML comments in markdown that look like old content (not intentional template)
- Code comments referencing identifiers that no longer exist (reuse C5 data)
- Flag: **COMMENT-ROT**

## Efficiency rules

- **Parallelism**: checks that share no scan results can be dispatched as parallel subagents. Independent: C4, C8, C11, C12, C13. Sequential required: C5 before C14 (share scan), C7 before C9 (share scan).
- **Dispatched agents are scan-only**: any dispatched agent MUST return findings text only, never call `Edit`/`Write`/`Bash` that mutates the project. All output comes back to this skill for unified reporting.
- **Truncation**: if a single check exceeds 50 findings, stop it early, report the first 50, note `truncated — N more found`.
- **No scratch files in the project**: findings accumulate in conversation context. If context pressure becomes real on very large repos, dispatch each check to a separate subagent and have it return a summary (not raw findings) — this trades precision for context budget, which is the right trade at scale. **Never write a scratch/findings file into the user's project** — the audit is report-only and does not modify project files.

## Output

```
## Repo Hygiene Audit — <project name>
**Date:** <YYYY-MM-DD>
**Scope:** <whole / recent-200 / subdir / diff>
**Scanned:** <N> files | **Type:** <doc-only / code-only / mixed>

### 🔴 High signal (P0)
- [BROKEN-REF] docs/setup.md:42 — link to `docs/install.md` (not found); rename candidate: `docs/installation.md`
- [CONFLICT] config.yaml:5 + retry.ts:18 — MAX_RETRY differs (3 vs 5)

### 🟡 Medium signal (P1)
- [DEAD-CODE] utils/legacy.ts — `parseOldFormat` has 0 references; not in do-not-touch set
- [UNUSED-DEP] package.json — `chalk` has 0 imports in src/
- [DRIFT] README.md:15-22 — directory tree shows `src/utils/` but actual path is `lib/utils/`

### 🟢 Low signal (P2)
- [BLOAT] src/api/handler.ts — 680 lines, max nesting 6
- [TEST-SHALLOW] tests/api.test.ts — 78% mock setup, avg 1.2 assertions per test

### Summary
| Priority | Found |
|----------|-------|
| P0 | N |
| P1 | N |
| P2 | N |
```

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Auto-fixing "safe" findings | Report only. User decides. |
| Running every day | Weekly / monthly. Daily is noise. |
| Skipping do-not-touch set for dead-code detection | Framework convention paths and package.json `exports` are NOT dead; never flag for removal. |
| Collapsing audit + fix into one session | Keep separate. The review pass is a Layer 5 defense. |
| Auto-editing files in dispatched subagents | Dispatched agents are scan-only; edits only happen in explicit follow-up work. |

## When NOT to use

- No `.harness/` → `bootstrap-new-project` or `adopt-into-existing-project`
- Mid-feature crunch — audits need attention-bandwidth you don't have
- Within 1 week of a clean audit — too soon

## Relationship to the standalone repo-hygiene-audit

A longer, more implementation-detailed version of this checklist exists at `claude-code/skills/repo-hygiene-audit/` (in the parent repo) as a standalone skill. This bundle version is intentionally more compact and aligns with bundle-wide conventions (report-only, no auto-fix, `.harness/` sediment awareness). If you need regex patterns and detailed auto-fix logic, see the standalone; if you're inside a harness-disciplined project, use this one.
