# Six-Layer Failure Model for AI Coding

AI-assisted coding fails in six distinguishable ways. Knowing which layer a problem lives in tells you whether harness engineering can fix it — and how.

## The layers

```
Layer 6  Workflow & system traps        ← harness territory
Layer 5  Human atrophy                   ← harness CANNOT fix; can worsen
Layer 4  Codebase entropy                ← harness + human judgment
Layer 3  Verification illusion           ← strong harness territory
Layer 2  Intent loss                     ← strong harness territory
Layer 1  Generation unreliability        ← mostly model-bound
```

Lower layers are per-interaction failures; higher layers are accumulation over time. **Most teams focus on layers 1-3 and underinvest in 4-6, where long-term damage accrues silently.**

---

## Layer 1 — Generation unreliability

The AI's single-shot output isn't trustworthy.

**Typical modes:**
- **Confidence-competence inversion** — it sounds more certain when wrong, because wrong answers are often high-frequency training patterns while right answers are codebase-specific.
- **Happy-path bias** — writes the main flow well, systematically ignores error paths, empty cases, concurrency, edge values unless prompted.
- **Plausible-fake** — cites functions, APIs, imports that look right but don't exist.
- **Mode collapse** — regresses to generic middle-of-distribution answers that ignore your project's specifics.

**Harness can:** structured output validation, forced grep-before-cite, required edge-case enumeration.
**Harness cannot:** cure confidence-competence inversion — only make AI confidence cheaper via evidence requirements.

---

## Layer 2 — Intent loss

What you meant doesn't survive the trip into the AI's working context.

**Typical modes:**
- **Rule conflict** — `CLAUDE.md` says A, a skill says B, memory says C; AI picks one silently.
- **Stale rules** — rules reference functions, files, or APIs that no longer exist.
- **Cargo-cult rules** — rules copied from elsewhere, not actually needed in this project.
- **Signal-noise decay** — as rules multiply, the important ones compete with the trivial for attention, and the important ones lose.

**Harness strongly can:** rule layering, skill-based activation (don't load everything every session), rule GC, conflict detection.
**Harness cannot:** tell you whether a rule is load-bearing — that's still human judgment.

---

## Layer 3 — Verification illusion

You think you verified; you didn't.

**Typical modes:**
- **Self-verification trap** — AI reviewing its own code with the same blind spots that produced it.
- **False-green tests** — tests pass because they mock everything or assert nothing meaningful.
- **Sample-size-1** — ran it once, worked, declared done.
- **Review fatigue** — after 30 minutes of reviewing AI diffs, bug-detection drops sharply without you noticing.
- **Plausibility bias** — AI code is structurally reasonable-looking, so reviewers skim.

**Harness strongly can:** forced test execution via hooks, independent review agents (not the same agent that wrote), evidence-required-before-claim patterns.
**Harness cannot:** force you to actually read a diff. That's Layer 5.

---

## Layer 4 — Codebase entropy

Local changes are fine; the whole rots.

**Typical modes:**
- **Quality corruption** — each change is locally optimal, globally chaotic.
- **Convention drift** — naming, imports, error handling slide across sessions.
- **Coherence erosion** — abstraction layers muddle (business logic in utils, util logic in handlers).
- **Lesson evaporation** — hard-won insights from one feature don't carry to the next.
- **Dead context** — `CLAUDE.md` grows monotonically; nothing gets deleted.

**Harness can:** periodic audits (`audit-harness`, `audit-repo-hygiene`), lint/format hooks, memory hygiene, lessons log.
**Harness cannot:** fix architecture taste — the "should this module exist?" judgment stays human.

---

## Layer 5 — Human atrophy

**The most insidious layer, and the only one harness cannot fix.**

**Typical modes:**
- **Taste anchoring** — if 99% of code you see is AI-written, your sense of "good" drifts toward the AI median.
- **Review skill ≠ writing skill** — people who write well don't automatically review well; this is a separate, learnable muscle that atrophies without deliberate practice.
- **Trust miscalibration** — you don't have a concrete map of "where AI is reliable vs where I must check."
- **Onboarding inversion** — new team members start with AI assistance before learning fundamentals; they become good prompters but never grow judgment.

**Harness cannot fix this layer.** Worse, **better harness can accelerate atrophy** — smoother AI means less human engagement means faster degradation. Conscious harness design preserves human friction at key points:

- Mandatory read-every-diff before approval
- Solo-coding slots (no AI) scheduled weekly
- Architectural review checkpoints with no AI shortcut
- Periodic hand-written bootstrapping of small features

This is the **hardest discipline to maintain** because there's always a short-term reason to skip it.

---

## Layer 6 — Workflow & system traps

Operational problems that aren't "bad code" but still cost you.

**Typical modes:**
- **Speed illusion** — code volume ≠ progress. AI output still needs review, verify, integrate; net advance is often 30% of gross.
- **Work amplification** — AI generates review burden faster than you can process it; you become its async assistant.
- **Feedback-loop latency** — slow verification means AI drifts further before you catch it.
- **Session sprawl** — state across multiple terminals, worktrees, files; no single source of truth.
- **Self-referential degradation** — AI writes code → AI reviews code → AI writes docs → AI reads docs; quality asymptotes to mediocrity without a mandatory human link.

**Harness can:** session management, cost tracking, permission gates, hook-based automation.
**Harness cannot:** prevent the self-referential loop if no human stays in the chain.

---

## Using this model

When something feels wrong, ask: **which layer?** The answer determines whether to reach for a harness tool, a process change, or a human investment.

| If the problem is in... | Reach for |
|------------------------|-----------|
| Layer 1 (bad output) | Tighter prompts, forcing functions, verification gates |
| Layer 2 (drifting rules) | `audit-harness` — GC, consolidate, simplify |
| Layer 3 (false confidence) | Independent review, hooks that force verification |
| Layer 4 (decaying codebase) | `audit-repo-hygiene`, lessons log, periodic cleanup |
| Layer 5 (your own atrophy) | Solo coding slots, read every diff yourself, don't delegate judgment |
| Layer 6 (workflow friction) | Workflow redesign, harness config tuning |

The layers interact — a Layer 2 rule-sprawl problem often manifests as Layer 4 convention drift. **Fix at the highest layer you can reach;** downstream layers often self-heal when the root cause is addressed.

## The one layer no one talks about

Layer 5 is underdiscussed because it's uncomfortable: admitting that better tooling might make you worse at your craft. Yet it's the layer with the longest time-constant of damage — by the time you notice taste anchoring or review atrophy, you've been drifting for months.

**If you take one thing from this model:** Layer 5 is real, it's inevitable without conscious resistance, and no harness will save you from it. Build human friction into your process on purpose.
