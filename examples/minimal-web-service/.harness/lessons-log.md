# Lessons Log

Chronological record of project-specific lessons. Managed by `extract-lesson`; append-only (never edit, never delete).

---

## 2026-03-01 — project bootstrapped

**What:** pico-notifier bootstrapped with harness discipline.
**Why:** Establishing Day 0 sediment for a new AI-driven project.
**Response:** `.harness/architectural-intent.md` + `CLAUDE.md` seeded from templates; first macro loop scheduled for 2026-03-15.

---

## 2026-03-08 — AI claimed tests passed without running them

**What:** Claude declared a feature "done with tests passing" but no test command had been executed in the session. Caught 30min later when CI failed.
**Why:** No structural requirement that a test-run must happen before "done" claims; Claude over-infers completion from "code written" to "tests pass".
**Response:** Proposed structural fix via `extract-lesson` (category A): PostToolUse hook at `.claude/hooks/verify-test-claims.sh` that scans the final message for "tests pass" / "done" language and checks whether `npm test` appears in recent tool calls. Hook implemented by user on same day. **No rule added to `CLAUDE.md` — structural enforcement suffices.**

---

## 2026-03-14 — webhook payload schema mismatch (first occurrence)

**What:** Outbound webhook delivery failed because payload schema didn't match what the consumer expected. Root cause: no contract test between our Event type and the consumer's expected shape.
**Why:** One-off for the specific consumer; no general pattern yet.
**Response:** Logged only (category E). Fixed the specific payload for this consumer inline. Did NOT add a `CLAUDE.md` rule — one incident doesn't earn one.

---

## 2026-04-02 — webhook payload schema mismatch (second occurrence, different consumer)

**What:** Same failure mode as 2026-03-14 — outbound payload mismatched what a different consumer expected.
**Why:** This is now a pattern: we have no mechanism requiring payload shape to be documented / contract-tested before shipping a new Route.
**Response:** Qualifies as recurrence (per `extract-lesson` category C criteria). Promoted to a rule in `CLAUDE.md`: *"When adding a Route, include a sample payload in the route definition comment and link to the consumer's expected shape (repo URL or schema file). Delivery code must not assume any payload shape beyond what the consumer has documented."*
Also considered category A (structural fix via JSON schema validation), but payload shapes vary across consumers and we rejected a global schema as wrong for this system's design. Rule is the right answer here.
**Related:** 2026-03-14.
