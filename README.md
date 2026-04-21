# harness-steward

**Project-level AI coding discipline for Claude Code and Codex.**

A skill bundle that installs discipline for projects where AI is the main coder and humans are architects, reviewers, and verifiers. `harness-steward` activates at key project moments — when a requirement is proposed, before implementation starts, after a task completes, when a bug surfaces, at the end of a feature, and on a periodic cadence — consulting project-specific sediment (architectural intent, lessons-log) and surfacing project-aware findings. Per-feature methodology (brainstorming, TDD, systematic-debugging) is the domain of complementary bundles like [superpowers](https://github.com/obra/superpowers); `harness-steward` hands off to those with project context pre-loaded.

## What problem this solves

When AI writes most of the code, individual commits look fine but over weeks the codebase drifts:

- Rules accumulate in `CLAUDE.md` until nothing is load-bearing
- Conventions slip between sessions
- Hard-won lessons from one feature get lost before the next
- You stop recognizing code quality because you're always seeing the AI median
- Configuration (hooks, skills, memory) rots without anyone noticing

`harness-steward` gives you **structural defenses** against these failures — not "remember to do X," but "the system makes X happen" or "the system makes it obvious when X is missing."

## What you get

Nine skills organized by when they activate:

**Setup (one-time, per project)**

| Skill | When |
|-------|------|
| `bootstrap-new-project` | Day 0 of a new AI-driven project |
| `adopt-into-existing-project` | Bringing discipline to an existing codebase |

**Workflow (activates at common development moments)**

| Skill | When |
|-------|------|
| `evaluate-requirement` | New requirement / PRD proposed — GO / MODIFY / DEFER / REJECT decision support against architectural intent + history |
| `scope-implementation` | About to start implementing — loads architectural boundaries, past lessons, existing conventions before code is written |
| `review-task` | Task complete, before commit — scans diff against project sediment; flags drive-by, no-evidence, boundary violations, lesson candidates |
| `debug-with-history` | Bug / incident encountered — frames the problem, searches lessons-log for precedent, checks architectural-intent for boundary manifestations |

**Periodic macro (weekly / monthly)**

| Skill | When |
|-------|------|
| `audit-harness` | Is your harness artifact stack (rules, intent, lessons-log, settings) still matched to reality? |
| `audit-repo-hygiene` | Is the codebase itself decaying across 14 dimensions (dead code, drift, duplication, dep bloat, etc.)? |

**Lesson capture**

| Skill | When |
|-------|------|
| `extract-lesson` | End of a feature / after a fix: decide whether the learning becomes a structural defense, an architectural intent update, a rule in `CLAUDE.md`, a logged history entry, or nothing |

Plus 9 reference documents, 2 templates, and 1 worked example in `references/`, `templates/`, and `examples/`.

## What this is NOT

- **Not a replacement for superpowers.** `harness-steward` governs project-level discipline; superpowers governs per-feature development. Use both if you can.
- **Not a linter or formatter.** `harness-steward` governs the *process* around AI coding, not code style.
- **Not opinionated about your stack.** Skills are stack-agnostic. Project-specific conventions get sedimented **into your project**, not into the bundle.

## The universal vs. sediment architecture

`harness-steward` draws a hard line between **universal content** (lives in the bundle, rarely changes) and **project-specific sediment** (AI writes into each project under `.harness/`):

```
your-project/
├── CLAUDE.md                        ← lean, project-specific rules
└── .harness/
    ├── architectural-intent.md      ← written Day 0 by you, referenced forever
    └── lessons-log.md               ← grown by extract-lesson over time
```

The bundle never writes to itself. Your project doesn't depend on the bundle at runtime — only when you actively invoke a skill. This means:

- Upgrading the bundle never breaks your projects
- Your sediment survives bundle changes
- The bundle stays small because project-specific knowledge doesn't pollute it

## Installation

### Via Claude Code marketplace (recommended)

```
/plugin marketplace add YEriin/harness-steward
/plugin install harness-steward
```

The manifest at `.claude-plugin/plugin.json` + the marketplace listing at `.claude-plugin/marketplace.json` make this a standard Claude Code plugin install.

### Via Codex marketplace

Codex has its own plugin format. This repository now exposes a Codex marketplace at `.agents/plugins/marketplace.json` and a Codex plugin at `plugins/harness-steward/.codex-plugin/plugin.json`.

For a published repo:

```bash
codex marketplace add YEriin/harness-steward
```

Then install `harness-steward` from Codex's plugin UI.

For local development against a clone of this repository:

```bash
codex marketplace add /absolute/path/to/harness-steward
```

This preserves bundle structure, so skills that rely on `templates/` and `references/` continue to work.

### Direct skill install for Codex (fallback only)

Codex also supports user-level skills in `~/.agents/skills/`, but this bundle is not designed to be fully installed that way. Several skills depend on bundle-relative assets in `templates/` and `references/`, so direct skill install is a degraded mode suitable only if you knowingly want the asset-free subset of skills.

### Local development (before publishing)

Symlink the bundle into Claude Code's plugin cache:

```bash
# Clone
git clone https://github.com/YEriin/harness-steward ~/harness-steward

# Option 1: direct skill install (simplest; some degradation on template/reference lookups)
for dir in ~/harness-steward/skills/*/; do
  ln -sfn "$dir" ~/.claude/skills/$(basename "$dir")
done

# Option 2: proper plugin install (preserves bundle structure)
mkdir -p ~/.claude/plugins/cache/local
ln -sfn ~/harness-steward ~/.claude/plugins/cache/local/harness-steward
# Then add an entry to ~/.claude/plugins/installed_plugins.json — see CONTRIBUTING.md
```

For Codex local development, prefer:

```bash
codex marketplace add /absolute/path/to/harness-steward
```

## Quick start

### Setup (once per project)

```
/bootstrap-new-project       # new projects — writes .harness/, minimal CLAUDE.md
/adopt-into-existing-project # existing codebases — staged introduction
```

### Per development activity

```
/evaluate-requirement   # "should we add X?" → GO / MODIFY / DEFER / REJECT
/scope-implementation   # "let's build Y" → loads project context before coding
/review-task            # "done, ready to commit" → diff-level project-aware review
/debug-with-history     # "bug in production" → check history + intent before debugging
/extract-lesson         # "feature done" → should this learning be preserved?
```

Each activates at its natural moment and — where applicable — hands off to superpowers methodology skills with project context pre-loaded.

### Periodic (weekly / monthly)

```
/audit-harness          # harness artifact health: rules, intent, lessons-log, settings
/audit-repo-hygiene     # codebase decay: 14 dimensions across P0/P1/P2
```

Run weekly for active projects, monthly for stable ones. **Don't run both every day** — it's noise, not signal.

## Philosophy

Full detail in `references/`. Short version:

- **AI coding fails in six distinguishable layers** (generation, intent, verification, entropy, human atrophy, workflow). See `references/six-layer-failure-model.md`.
- **Development runs on three nested loops** (macro, meso, micro). `harness-steward` owns the macro loop and provides activation points at meso-loop boundaries. See `references/three-loop-workflow.md`.
- **Structural over behavioral.** Rules in `CLAUDE.md` hope; hooks / lint / CI enforce. This bundle prefers structural defenses and skills that activate at moments over regulations that depend on AI remembering. See `references/harness-principles.md`.
- **Sediment must be discoverable.** Every file the bundle writes into a project must have an ambient-discovery mechanism (e.g. `CLAUDE.md` pointer) — otherwise it's write-only waste. See the ambient-discovery rule in `references/harness-principles.md`.
- **Harness cannot cure human atrophy.** Better harness can accelerate atrophy if humans disengage. Conscious design preserves human friction at key points — read-every-diff, solo-coding slots, architectural review checkpoints.

## Relationship to superpowers

| | superpowers | harness-steward |
|-|-----------|---------------|
| Time scale | Hours, days | Weeks, months, project lifetime |
| Core question | "How do I develop this feature well?" | "How do I keep this project healthy?" |
| Typical trigger | "Let's build the auth system" | "Is the project's harness up to date?" |

They don't depend on each other. If both are installed, Claude routes naturally by context. See `references/relationship-to-superpowers.md`.

## Status

**v0.2.0 — beta.** All 9 skills are written and individually TDD-validated (baseline-vs-skill subagent scenarios). Each skill passed a cross-skill consistency pass against the prior set; contradictions surfaced during reviews were resolved before release. v0.2.0 incorporates the first real-world feedback pass (Orbito, 2026-04-21) as structural hardening — see `CHANGELOG.md` for the four error classes it closes and the scan-output-schema contract introduced to prevent regression. **Still pre-1.0**: one validation report is a signal, not a full gate; more real-world runs across varied stacks are the remaining path to v1.0.

See `CHANGELOG.md` for what shipped. See `CONTRIBUTING.md` for development notes. See `references/finding-tag-glossary.md` if you want to understand the structured tags skills emit in their reports.

Feedback, issues, and PRs welcome.

## License

MIT. See `LICENSE`.
