# Developer Discipline for AI-Assisted Coding

When AI is the primary coder, your job changes. This document captures the role shift and the core disciplines that make the new role work.

## The role shift

You are no longer a **writer**. You are an **architect + reviewer + verifier**.

These three functions always existed in traditional coding — but they were implicit, fused with the act of writing. When AI writes, they have to become **explicit**, and no one else will do them for you.

The most common failure of engineers adopting AI-heavy coding: treating the AI as a faster version of themselves, and never completing this role shift. Symptoms:

- Code volume grows but your confidence shrinks
- "Looks reasonable" code accumulates, then reveals architectural drift weeks later
- You keep nodding at AI's "done" claims; production later shows you they weren't
- Your own judgment quietly erodes as you increasingly rely on AI's confidence

## Three iron rules

### 1. Specify at "outcome + constraint" altitude, not implementation

- **Too low** (*"write a loop that..."*) — you become a slow compiler; AI's capacity is wasted
- **Too high** (*"build auth"*) — AI makes a dozen unauthorized decisions in 30 minutes
- **Right altitude** — *"Input X → output Y; must not modify Z; must handle W edge case"*

Sweet spot: specific enough to constrain, open enough to let AI find a good solution.

### 2. Small reviewable chunks; read the diff, not the summary

Hard rule: **if a single interaction produces a diff you can't read end-to-end in 2 minutes and understand why each line is there, you have already lost control — you just haven't noticed yet.**

AI's self-reported summary ("I updated the regex to handle X") is self-report, not evidence. The diff is truth. This is the hardest discipline because there's constant social pressure to say "looks good, next."

### 3. Verification is yours

"Tests passed" ≠ "the feature works." Tests passing means the assertions you wrote were satisfied. It does not mean the assertions themselves are right.

- UI change → open the browser, use it
- API change → make a real request, read the response
- Data change → query before and after, confirm state

**Assume AI's self-reported completion overstates by 10-20%.** Not because AI is lying — because it observes "action taken" and infers "done" more readily than it should.

## Meta discipline: keep your hands dirty

If you stop writing code entirely, your review skill degrades within months. Not because you become less capable overall — because your taste reference point slides toward the AI median. A good ratio:

> **AI writes ~70%, you write the tricky / opinionated / novel 30%.**

Not efficiency-optimal. Capability-preservation-optimal. This is a conscious investment in Layer 5 resistance (see `six-layer-failure-model.md`).

## Crash scenarios: where careless adoption ends projects

### Irreversible operations

DB migrations, force pushes, `rm -rf`, external API calls with side effects. **You review before, not after.** Non-negotiable — the blast radius makes "oops, revert" impossible.

### Architecture decisions

"Should this live in the auth layer or the service layer?" "Do we need this abstraction?" AI's architectural instinct is **sometimes okay, sometimes terrible, always unreliable**. Ownership of architecture stays with you.

### Long-session context drift

Past ~2-3 hours of continuous dialogue, AI starts forgetting early constraints, contradicting itself, drifting from rules you established an hour ago. **Reopen the session rather than push through.** Context compaction is not free.

## How this connects to the rest of the bundle

- **Role shift** → the reason harness exists in the first place
- **Three iron rules** → per-interaction discipline no harness can do for you
- **Meta discipline** → Layer 5 (human atrophy) resistance
- **Crash scenarios** → why `bootstrap-new-project` sets up safety rails Day 0

Treat this doc as the **mental model** you bring to every interaction with the AI. Skills enforce specific processes; this document shapes the worldview those processes operate inside.
