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
- **Key files**: manifests (any ecosystem manifest — `package.json`, `go.mod`, `pyproject.toml`, `Cargo.toml`, `pom.xml`, `Gemfile`, `composer.json`, `deno.json`, etc.; the list is illustrative, treat any dep/build-declaring file as a manifest), config (`.env*`, `*.yaml`, `*.toml`), tests (`test*/`, `*_test.*`, `*.test.*`, `*.spec.*`)

### Scope is reality-time, not author-time

Before any dispatch, build two data structures and pass them to every scan agent. Author-time defaults (`src/`, `e2e/`, `tests/`, `node_modules`) are a floor, not the truth — real projects have non-standard layouts.

1. **Ignore patterns** — read `.gitignore` and `.git/info/exclude`; merge with hard-coded defaults (`node_modules`, `.venv`, `vendor`, `dist`, `build`, `.git`). **No finding may target a path matching these patterns.** This kills the "X should be gitignored" false-positive class entirely.
2. **Source roots** — do NOT assume `src/` is the only source root. Enumerate every directory that carries an ecosystem manifest (any dep/build-declaring file — examples in Phase 0 detection above; the list is not exhaustive — when a project uses an unfamiliar ecosystem, treat its manifest the same way). Honor workspace/monorepo files (`pnpm-workspace.yaml`, `go.work`, root `Cargo.toml` workspace table, `lerna.json`, `nx.json`, `turbo.json`, Gradle `settings.gradle*`, Maven reactor `pom.xml`, etc.) to expand the set. Record the union as the `source_roots` list. Every grep for imports, dep usage, or symbol references MUST iterate over `source_roots`, not hard-coded `src/`+`e2e/`. Full pattern + language-agnostic extension guidance: `../../references/scan-output-schema.md`.

### Scope gate

If `>500 files` (after applying the ignore patterns above) and no scope arg was given, ask the user:
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

## Phase 4 — Verification pass (mandatory, before emit)

Scan agents produce findings; this phase turns them into truth. The **full contract** — line format, probe tables, neutralization rules — lives in `../../references/scan-output-schema.md`. Skill-level summary of what MUST run before any finding is emitted:

- **V1. Identifier verification** — any finding with `identifier:<name>` at `path:line` requires a grep at `line ± 5`. No hit → drop the name (or the finding). *Why:* blocks correct-anchor + invented-name hallucinations.
- **V2. Prescriptive-claim probes** — descriptors like "should be gitignored", "unused dep", "file/dir missing", "symbol not found" each require a reality probe (`git check-ignore`, cross-source-root grep, `test -e`, whole-repo grep). Probe passes → drop. *Why:* blocks scope-blind false positives against `.gitignore` and non-standard source roots.
- **V3. Severity-language constraint** — charged adjectives (`retired`, `broken`, `deprecated`, `highest-impact`, etc.) require `evidence:<url|cmd-output>`. Missing evidence → neutralize the descriptor; severity is set by the aggregator, not the sub-agent. *Why:* blocks P0-inflation via editorial language.
- **V4. Out-of-scope reality pass** — **P0 findings only**, only when the descriptor references a literal token (config key, env-var, model ID, port): run one unfiltered `grep -rn` across the whole repo. Extra hits → annotate `scope-incomplete` and list them. *Why:* catches drift the scoped scan couldn't see. Bounded to P0 to keep the cost of the one O(repo) probe controlled.

Concrete probe tables, neutralization mappings, and the reference aggregator sketch are in the schema doc — don't duplicate them here.

## Efficiency rules

- **Parallelism**: checks that share no scan results can be dispatched as parallel subagents. Independent: C4, C8, C11, C12, C13. Sequential required: C5 before C14 (share scan), C7 before C9 (share scan).
- **Dispatched agents are scan-only**: any dispatched agent MUST return findings text only, never call `Edit`/`Write`/`Bash` that mutates the project. All output comes back to this skill for unified reporting.
- **Structured output contract**: scan agents MUST emit findings in the line format from `../../references/scan-output-schema.md` — `[TAG] path:line — short-descriptor — identifier:<name?> evidence:<regex|cmd|url>`. Free-text narrative in the findings stream is banned; narrative goes in the aggregator's top-level summary only. This is what makes Phase 4 verification tractable.
- **Truncation**: if a single check exceeds 50 findings, stop it early, report the first 50, note `truncated — N more found`.
- **No scratch files in the project**: findings accumulate in conversation context. If context pressure becomes real on very large repos, dispatch each check to a separate subagent and have it return a summary (not raw findings) — this trades precision for context budget, which is the right trade at scale. **Never write a scratch/findings file into the user's project** — the audit is report-only and does not modify project files.

## Output

```
## Repo Hygiene Audit — <project name>
**Date:** <YYYY-MM-DD>
**Scope:** <whole / recent-200 / subdir / diff>
**Scanned:** <N> files | **Type:** <doc-only / code-only / mixed>
**Source roots:** <detected list> | **Ignore patterns:** .gitignore + defaults

### 🔴 High signal (P0)
- [BROKEN-REF] docs/setup.md:42 — link to `docs/install.md` — identifier:`docs/install.md` evidence:`test -e` → missing; rename candidate: `docs/installation.md`
- [CONFLICT] config.yaml:5 + retry.ts:18 — MAX_RETRY differs (3 vs 5) — evidence:`grep -n MAX_RETRY`
- [CONFLICT] deploy/.env.production:14 — AI_MODEL differs from CHANGELOG-documented value — evidence:CHANGELOG.en-US.md:74 [scope-incomplete: also present in CONTRIBUTING.md:104, docs/ops/infrastructure.md:176]

### 🟡 Medium signal (P1)
- [DEAD-CODE] utils/legacy.ts:12 — identifier:`parseOldFormat` evidence:`grep -rn 'parseOldFormat'` → 0 refs outside definition; not in do-not-touch set
- [UNUSED-DEP] package.json — identifier:`chalk` evidence:`grep -rn 'chalk'` across source_roots → 0 imports
- [DRIFT] README.md:15-22 — directory tree references `src/utils/` evidence:`test -e src/utils` → missing; actual path is `lib/utils/`

### 🟢 Low signal (P2)
- [BLOAT] src/api/handler.ts — 680 lines, max nesting 6 — evidence:`wc -l` + AST
- [TEST-SHALLOW] tests/api.test.ts — 78% mock setup, avg 1.2 assertions per test

### Summary
| Priority | Found | Verified | Dropped in Phase 4 |
|----------|-------|----------|---------------------|
| P0 | N | N | N |
| P1 | N | N | N |
| P2 | N | N | N |

Dropped categories (top causes):
- identifier not found at claimed location: N
- path already gitignored: N
- dep imported in non-standard source root: N
- charged adjective without evidence: N
```

The `identifier:` and `evidence:` fields are part of the emit contract, not optional decoration — see `../../references/scan-output-schema.md`. The Dropped-categories block exists so the user can see what the verification pass caught; silently dropping findings would obscure quality regressions.

## Anti-patterns

| Failure mode | Correct behavior |
|--------------|-----------------|
| Auto-fixing "safe" findings | Report only. User decides. |
| Running every day | Weekly / monthly. Daily is noise. |
| Skipping do-not-touch set for dead-code detection | Framework convention paths and package.json `exports` are NOT dead; never flag for removal. |
| Collapsing audit + fix into one session | Keep separate. The review pass is a Layer 5 defense. |
| Auto-editing files in dispatched subagents | Dispatched agents are scan-only; edits only happen in explicit follow-up work. |
| Skipping Phase 4 verification | V1–V4 are mandatory before emit. The four failure classes this skill was hardened against are all verification-pass-absent failures. |
| Flagging paths already covered by `.gitignore` | Phase 0 reads `.gitignore`; findings targeting ignored paths MUST be filtered before emit. |
| Grepping only `src/`+`e2e/` for unused deps | Phase 0 detects all source roots (every dir with a manifest, plus workspace entries); V2 iterates over the detected set. |

## When NOT to use

- No `.harness/` → `bootstrap-new-project` or `adopt-into-existing-project`
- Mid-feature crunch — audits need attention-bandwidth you don't have
- Within 1 week of a clean audit — too soon

## Relationship to the standalone repo-hygiene-audit

A longer, more implementation-detailed version of this checklist exists at `claude-code/skills/repo-hygiene-audit/` (in the parent repo) as a standalone skill. This bundle version is intentionally more compact and aligns with bundle-wide conventions (report-only, no auto-fix, `.harness/` sediment awareness). If you need regex patterns and detailed auto-fix logic, see the standalone; if you're inside a harness-disciplined project, use this one.
