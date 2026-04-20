# Three-Loop Workflow

AI-assisted development runs on three nested loops at different time scales. Each has its own discipline and its own failure modes.

```
┌────────────────────────────────────────────────────┐
│ Macro loop  (weekly / monthly)                      │
│ Project health, harness maintenance                 │
│ ┌────────────────────────────────────────────────┐ │
│ │ Meso loop  (per feature / PR)                   │ │
│ │ Spec → plan → execute → verify → ship           │ │
│ │ ┌────────────────────────────────────────────┐ │ │
│ │ │ Micro loop  (hours)                          │ │ │
│ │ │ Single change → verify → commit              │ │ │
│ │ └────────────────────────────────────────────┘ │ │
│ └────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

---

## Micro loop

**Time scale:** minutes to hours.
**Question:** "Is this one change correct?"

**Composition:**
- Specify the small change
- AI executes
- You verify (read diff, actually test)
- Commit

**Harness tools:** existing dev tools (lint, test, format). This is where [superpowers](https://github.com/obra/superpowers) lives — it turns micro-loop discipline (TDD, debugging, verification) into reusable skills.

**`harness-steward` does not own this loop.** It assumes you have some methodology in place here.

---

## Meso loop

**Time scale:** hours to days.
**Question:** "Is this feature done well?"

**Composition:**
- Scope the feature (architectural intent check: does this belong?)
- Plan (break into micro loops)
- Execute micro loops
- Integrate and verify the whole
- **Extract lesson:** what did I learn that should outlive this feature?

**`harness-steward` contributes:** `extract-lesson` is meso-loop's closing skill. Superpowers handles the rest.

**Critical discipline:** don't skip extraction. Lessons not captured = lessons lost. But also don't codify everything — use `extract-lesson`'s decision tree.

---

## Macro loop

**Time scale:** weekly, monthly.
**Question:** "Is this project still healthy? Is my harness still right?"

**Composition:**
- Review architectural intent against current code
- Audit harness: are rules still load-bearing? Are there stale ones?
- Audit code: has hygiene decayed?
- Prune accumulated noise (dead rules, dead skills, dead context)
- Adjust harness based on observed pain from the week

**`harness-steward` owns this loop entirely:**
- `audit-harness` — harness / config / rules health
- `audit-repo-hygiene` — code / docs / artifact health
- (Occasionally) `adopt-into-existing-project` if you're adding discipline somewhere new

**This is the loop most teams skip, and the reason codebases rot under otherwise-healthy feature work.** Without a macro loop, all the discipline from meso and micro loops still leads to slow decay — because the discipline itself is never re-examined for drift.

---

## How to use this model

- **Starting a feature:** "I'm in a meso loop now. What's the spec? What's the lesson target?"
- **Something feels bloated:** "I need a macro loop. Schedule an audit."
- **A change hits a wall:** "Is this a micro-loop problem (this change is hard), a meso-loop problem (wrong scope), or a macro-loop problem (architectural drift)?"

**Mis-attributing a macro problem to a micro loop is the most common failure** — you keep trying to fix "this change" when the real issue is the project's harness is stale.

## Cadence recommendation

| Loop | Cadence |
|------|---------|
| Micro | Continuous while coding |
| Meso | Per feature / PR |
| Macro | Weekly for active projects; monthly for stable ones |

**Don't skip the macro loop because "everything is fine."** Things seem fine until they don't. Scheduling it as a regular slot (Friday afternoon, start of each sprint) prevents it from being forever-deferred into never.

## Relationship to bundles

| Loop | Primary bundle |
|------|----------------|
| Micro | superpowers (or your own workflow) |
| Meso | superpowers for most of it; `harness-steward` for `extract-lesson` at close |
| Macro | `harness-steward` |

Each loop has a different owner. Don't mix them — a macro-loop question doesn't need superpowers, and a micro-loop change doesn't need harness-steward.
