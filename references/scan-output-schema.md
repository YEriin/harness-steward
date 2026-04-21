# Scan-Output Schema for Dispatched Agents

Contract that scan-only sub-agents MUST follow when dispatched by bundle skills (`audit-repo-hygiene`, `audit-harness`, `review-task`, and any future scan skill). The schema makes findings **verifiable** instead of narrative, which is the only structural defense against the two failure modes that narrative findings suffer from:

- **Identifier fabrication** — the structural anchor (`path:line`) is correct but the attached name is generated
- **Severity inflation** — charged verdict words (`retired`, `broken`, `deprecated`, `highest-impact`) attach without evidence

## Why structural, not behavioral

Asking the sub-agent to "be careful" is a Layer 1/3 defense that fails at scale (see `six-layer-failure-model.md`). A structured output line with required fields is a Layer 4 defense: the aggregator can mechanically verify every finding before emitting it.

## The line format

Each finding emitted by a dispatched agent MUST be a single line matching this shape:

```
[TAG] path:line — short-descriptor — identifier:<name?> evidence:<regex|cmd|url>
```

Field rules:

| Field | Required when | Content |
|-------|---------------|---------|
| `[TAG]` | always | One of the tags in `finding-tag-glossary.md` |
| `path:line` | always when locatable; omit if repo-wide | Absolute-from-repo path; `:line` optional for file-level findings |
| `short-descriptor` | always | ≤ 120 chars; neutral language (see severity rules below) |
| `identifier:<name>` | when the finding names a specific symbol, dep, file, or config key | The exact string the finding asserts exists |
| `evidence:<…>` | when the descriptor uses a charged verb OR asserts a specific count OR references an external fact | A regex, a command output snippet, or a stable URL |

Free-text narrative (paragraphs, hedging, context prose) MUST NOT appear in the findings stream. Put any narrative in the top-level summary the aggregator writes, not in dispatched agent output.

## Required aggregator verification steps

The parent skill (aggregator) MUST run these probes before promoting a finding to the final report. V1–V3 are O(1) or O(findings) — cheap, run on every eligible finding. V4 is the one O(repo) probe and is **bounded to P0 literal-referencing findings** to keep total cost controlled; see V4 for the exact gating.

### V1 — Identifier verification

For every finding that carries `identifier:<name>` at a `path:line` anchor:

1. Grep `<name>` in `<path>` within `line ± 5`
2. Not found → **downgrade** the finding: drop the `identifier:` field and replace the descriptor with a generic one (`"a <kind> at this location"`), OR drop the finding entirely if the name was load-bearing
3. Never emit a finding whose named identifier was not verified to exist

This single step eliminates the most common hallucination pattern: correct anchor + invented name.

### V2 — Prescriptive-claim probes

For descriptors shaped like prescriptive verdicts, run the mapped probe. The **pattern** is what's normative; specific tool names appear only as examples. When a project uses an ecosystem not listed here, apply the pattern — don't refuse to probe because the row isn't written.

| Descriptor shape | Pattern | Drop on |
|------------------|---------|---------|
| "should be gitignored" | `git check-ignore <path>` (works for any language) | exit 0 |
| "file/dir missing" | `test -e <path>` (works for any language) | exists |
| "symbol not found" | Grep the symbol across the whole repo, not just the initial scope | any hit |
| "unused dep" | For each `source_roots` entry from Phase 0, grep the dep's import form(s) across all sources under it. Include any well-known dev/test-only import sites the ecosystem uses (e.g. JS `scripts/`, `tools/`, sibling `<pkg>/src/`; Go `_test.go`, `tools.go`; Python typing stubs; Rust `examples/`, `benches/`). Ecosystem-specific: derive from the manifest, not from a fixed list. | any hit |

**How to extend to a new ecosystem:** identify (a) the manifest file, (b) the conventional import/dependency syntax, (c) the convention for dev-only / test-only / examples-only code. Build one additional row privately in the aggregator; do not require a schema edit for every ecosystem. The goal is to avoid the failure mode where a probe silently skips because the ecosystem wasn't anticipated — when in doubt, fall through to "grep across every `source_roots` entry" as the universal fallback.

### V3 — Severity language constraint

The `short-descriptor` MUST be neutral unless backed by `evidence:`. Banned adjectives without evidence:

- `retired`, `deprecated`, `broken`, `removed`, `end-of-life`
- `highest-impact`, `critical`, `blocker` (as editorial judgments, not severity tags)
- `unused` (use V2 probe result instead)
- `retired model`, `insecure`, `vulnerable` (require URL to authoritative source)

If the agent wants to use one of these, it MUST attach `evidence:<url>` or `evidence:<cmd-output>`. The aggregator checks for the evidence field; if missing, it rewrites the descriptor to a neutral form:

| Banned | Neutral replacement |
|--------|--------------------|
| "retired" | "differs from current project value" |
| "broken" | "does not resolve" |
| "deprecated" | "not found in current docs" |
| "highest-impact" | (drop the qualifier) |
| "unused" | "no imports found in scanned roots" |

Severity tags (🔴 P0 / 🟡 P1 / 🟢 P2) are set by the aggregator based on the tag + probe results, never by the sub-agent directly.

### V4 — Out-of-scope reality pass (P0 config tokens only)

For each P0 finding whose descriptor references a config token, env-var name, model ID, port, or similar literal, the aggregator MUST run one unfiltered `grep -rn <literal>` across the repo. Hits in files not visited by the scan cause the finding to carry a `scope-incomplete` annotation and the enumerated locations.

This is scoped to P0 only because it is the one O(repo) probe; running it for every finding would blow the budget on a large repo.

## Scope-awareness obligations (Phase 0)

Before dispatching any scan agent, the aggregator MUST:

1. **Read `.gitignore` and `.git/info/exclude`** and merge those patterns into the exclusion filter. Hard-coded exclusions (`node_modules`, `vendor`, `dist`) are a floor, not a ceiling — the project's own ignore file is authoritative for "what is not source."
2. **Detect source roots dynamically.** Don't assume `src/` / `e2e/`. A "source root" is any directory that carries an **ecosystem manifest** — the file that declares the project's identity, dependencies, or build entry for that language/toolchain. Enumerate every directory containing such a file and treat each as a source root. Examples seen in the wild: `package.json`, `go.mod`, `pyproject.toml`, `Cargo.toml`, `requirements*.txt`, `setup.py`, `setup.cfg`, `pom.xml`, `build.gradle`, `build.gradle.kts`, `Gemfile`, `composer.json`, `mix.exs`, `Project.toml`, `deno.json`, `bun.lockb`, `*.cabal`, `dune-project`, `shard.yml`, `*.csproj` / `*.fsproj`. The list is **illustrative, not exhaustive** — any manifest-shaped file declaring dependencies or build config counts. Also honor workspace/monorepo files that expand the set (`pnpm-workspace.yaml`, `go.work`, root `Cargo.toml` workspace table, `lerna.json`, `nx.json`, `turbo.json`, Gradle `settings.gradle*`, Maven reactor `pom.xml`). When a project uses an ecosystem you don't recognize, **don't skip** — a file whose name or shape matches "manifest declaring deps/build" should be treated as one.
3. **Record both** in the dispatched agent's prompt so each agent scans the right scope, not the scope the skill author imagined.

Scope is **reality-time**, not author-time.

## Reference implementation sketch

An aggregator executing these rules looks roughly like:

```
# Phase 0
source_roots = detect_source_roots(cwd)
ignore_patterns = read_gitignore(cwd) ∪ hard_coded_defaults

# Dispatch scan agents with (source_roots, ignore_patterns)
findings = dispatch_parallel(checks, source_roots, ignore_patterns)

# Verification pass
for f in findings:
    if f.identifier and not grep_verified(f.identifier, f.path, f.line):
        f = degrade(f)  # drop identifier, generic descriptor, or drop finding
    if f.descriptor.has_banned_adjective() and not f.evidence:
        f.descriptor = neutralize(f.descriptor)
    if f.shape in PRESCRIPTIVE_SHAPES:
        if probe(f.shape, f).passes:
            drop(f)
    if f.severity == 'P0' and f.has_literal_token():
        extra = unfiltered_grep(f.literal)
        if extra - f.scanned_files:
            f.annotate('scope-incomplete', extra)

# Emit
report(findings)
```

## Applicability

Every bundle skill that dispatches sub-agents for **read-only scanning** is subject to this contract. Write skills (skills that produce or edit artifacts in the user's project) are out of scope — this document is specifically about scan-to-report pipelines.

Current adopters:
- `audit-repo-hygiene` — full adoption (identifier verification, prescriptive probes, scope-aware dispatch, P0 token sweep)
- `audit-harness` — partial (identifier verification for rule-staleness; prescriptive probes for dead-hook / stale-permission)
- `review-task` — partial (identifier verification for INTENT-VIOLATION and DRIVE-BY anchors)

Any future scan-class skill MUST cite this reference in its SKILL.md and state which of V1–V4 apply.

## History

Introduced after the 2026-04-21 Orbito feedback surfaced four verifiable error classes across one production run of `audit-repo-hygiene`: identifier fabrication on correct anchors, severity inflation, scope-blind false positives against `.gitignore`-covered paths and non-standard source roots, and recall gaps on P0 config tokens. Each class mapped to a structural gap that this schema closes.
