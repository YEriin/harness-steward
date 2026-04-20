# Relationship to superpowers

`harness-steward` covers project-level discipline. [superpowers](https://github.com/obra/superpowers) covers per-feature development methodology. They fit together well when both are installed, but neither depends on the other.

## Scope comparison

| | superpowers | harness-steward |
|-|-----------|-----------------|
| **Time scale** | Hours, days | Weeks, months, project lifetime |
| **Core question** | "How do I develop this feature well?" | "How do I keep this project healthy?" |
| **Loop ownership** (see `three-loop-workflow.md`) | Micro + most of Meso | Meso closing + Macro |
| **Typical skills** | brainstorming, writing-plans, TDD, debugging, review, verification, worktrees | bootstrap, audit, evaluate-requirement, scope-implementation, review-task, debug-with-history, extract-lesson |
| **Coupling** | Per-feature | Per-project, per-project-phase |
| **State** | Mostly in-conversation | Sediment in `.harness/` |

## Using both together

No integration is required. If both plugins are installed, Claude will route naturally based on context:

- "Let's start a new feature" → superpowers (brainstorming → writing-plans → TDD)
- "Is my project still healthy?" → `harness-steward:audit-harness`
- "I finished the feature — what did I learn?" → `harness-steward:extract-lesson`, then possibly back to superpowers (`finishing-a-development-branch`)
- "Start a new project" → `harness-steward:bootstrap-new-project`

Skill descriptions in both bundles are trigger-focused so routing happens by user intent, not by plugin namespace.

## Using harness-steward without superpowers

`harness-steward` works standalone. Where its skills suggest handing off to a `superpowers:...` skill, the hand-off is always optional — substitute your own workflow, another bundle, or a plain conversation.

## What harness-steward does NOT provide

For completeness, harness-steward explicitly leaves these to other tools or the user's workflow:

- Per-feature methodology (brainstorming, planning, TDD, debugging, code review, completion verification, worktree management)
- Generic code-quality linting / formatting (use your existing tools)
- Generation of feature code itself

## If you can only install one

- **Early project with AI-heavy development** → harness-steward gets the foundation (architectural intent, sediment, audits) right first
- **Mature project with many contributors and features in flight** → superpowers' per-feature discipline pays back fastest
- **Long-term AI-primary development** → both, since they protect against different failure modes

## Common confusion

**"Can I use `superpowers:writing-skills` to do my harness audits?"**
No. `writing-skills` is about creating skills (TDD for docs). `audit-harness` is about curating an existing corpus of harness artifacts in a live project.

**"Isn't `extract-lesson` just brainstorming in reverse?"**
No. `extract-lesson` applies a decision tree around "does this deserve to become a permanent rule, given that every rule dilutes attention from existing rules?" It's pruning, not generation.

**"Do I need harness-steward if I already have a well-curated `CLAUDE.md`?"**
Probably still yes — harness-steward helps keep it curated as the project evolves. Static curation decays; audit cycles prevent that.
