# Harness Engineering — Principles

## What "harness" means

The **harness** is the scaffolding around an LLM — tools, context, memory, hooks, skills, permissions, workflow — whose design makes desired behavior **structural** (system-enforced) rather than **behavioral** (model-dependent).

The mental shift:

> **Instead of hoping the AI does X every time, make X the only path.**

Harness engineering is the practice of shaping that scaffolding deliberately. It's where most of the leverage lives for reliability at scale — more than prompt engineering, more than model choice.

## Structural vs behavioral

| | Behavioral (fragile) | Structural (resilient) |
|-|-|-|
| Example | "Please run tests before claiming done" in CLAUDE.md | PostToolUse hook that runs tests and injects the result |
| Fails when | AI forgets, gets distracted, rationalizes | Never, unless the hook itself breaks |
| Noise cost | Adds to the rule pile in every conversation | Zero (invisible unless triggered) |
| Scaling | Degrades as rules multiply | Stable |

**Rule of thumb: anything you catch yourself wishing AI "would always" do is a candidate for structural enforcement.**

## MVH — Minimum Viable Harness

Harness is never built in one go. It **co-evolves with the project**, starting minimal. Over-engineering upfront produces cargo-cult harness — defenses against problems you haven't had.

**First-week harness** for a new AI-driven project:

- Architectural intent doc (1-2 pages)
- Skeleton `CLAUDE.md` (the 11-rule universal template baseline; resist adding project-specific rules on Day 0 — let recurrence earn them)
- Basic hooks: tests must pass, lint must pass, destructive ops require confirmation

Everything else is added **in response to observed pain**.

- ✅ *"Got bitten by X twice → add structural defense against X"*
- ❌ *"I can imagine X might happen → add defense preemptively"*

## Harness intensity × blast radius

**Not all code needs the same harness intensity.** Match friction to risk:

| Code type | Harness intensity | Example mechanisms |
|-----------|-------------------|-------------------|
| Exploratory / prototype | Low | Tests optional, no pre-commit hooks, speed first |
| Production feature | Medium | TDD expected, CI gates, PR review |
| Irreversible / high-risk (migrations, security, payments) | High | Plan mode mandatory, independent review agent, line-by-line human read |

Applying the same intensity everywhere either drowns the team in noise or waits for the inevitable disaster.

## The prime directive

**What you can make structural, don't leave as habit.**

- **Habit** — depends on the AI remembering and doing right each time; fails eventually at scale
- **Structure** — errors get blocked or surfaced; they don't accumulate silently

This is the single principle harness engineering exists to serve.

## The ambient-discovery rule

**Any sediment file the bundle writes into a project must have an ambient-discovery mechanism — otherwise it doesn't exist.**

Corollary of the prime directive. If a sediment file is only read when a harness skill is explicitly invoked, it's invisible during regular project work — which is when most code decisions are actually made. Sediment without ambient discovery is write-only waste.

**What counts as ambient discovery:**

- Reference from project `CLAUDE.md` (auto-loaded by Claude Code every session)
- SessionStart / PostToolUse / PreToolUse hooks that surface relevant content
- Reference from a commonly-triggered skill's description or routing body

**What does NOT count:**

- *"The user will tell AI to read it"* — behavioral, fragile, defeats the structural premise
- *"AI will notice the file exists"* — directory scans are not default behavior
- *"Documented in README"* — README is for humans, not for runtime AI context in a project

**Concrete application in this bundle**: when `bootstrap-new-project` or `adopt-into-existing-project` creates `.harness/architectural-intent.md` and `.harness/lessons-log.md`, the corresponding `CLAUDE.md` **must** include a "Project Context" section pointing at them. `audit-harness` Check 1f enforces the invariant; it will flag `MISSING-CONTEXT-POINTERS` if a project has the sediment but no pointer path.

**Generalizable**: extending harness with new sediment artifacts without simultaneously designing the discovery path creates silent-decay surface. Every new file written into a project is a promise that AI will read it later — a promise you must wire up structurally, not leave to vibes.

## The paradox

Better harness creates smoother AI interaction. Smoother interaction reduces human engagement. Reduced engagement accelerates Layer 5 atrophy (see `six-layer-failure-model.md`).

**Deliberate harness design preserves human friction at specific points:**

- Mandatory diff reads before approval
- Solo-coding slots (no AI) scheduled weekly
- Architectural review checkpoints with no AI shortcut
- Blast-radius gates requiring explicit human confirmation

This is the one thing harness engineering **cannot cure by adding more harness**. The solution is **removing smoothness on purpose** at the right places.

## How harness relates to adjacent concepts

| Concept | Relation to harness |
|---------|---------------------|
| **Prompt engineering** | Subset — harness includes system prompts, but also everything beyond |
| **Model engineering** | Orthogonal — harness operates on top of whatever model |
| **DevOps / CI** | Overlap — CI hooks are a form of harness; harness extends further (context, memory, skills) |
| **Agent frameworks** | Harness is the philosophy; frameworks are implementations |

## One-sentence summary

**Harness engineering is where you turn "AI should do X" into "AI can only do X" — and where you consciously keep the humans in the loop so they stay sharp.**
