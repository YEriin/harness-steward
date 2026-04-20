# Rules

## Project Context

This project uses harness discipline. Two project-specific artifacts carry authoritative context beyond this file:

- **`.harness/architectural-intent.md`** — defines what pico-notifier IS / IS NOT, key abstractions (Event / Route / Delivery) and explicit "does not know about" coupling constraints. **Read this before proposing module boundary changes, cross-module imports, or new abstractions.** Surface conflicts with stated intent rather than silently violating.
- **`.harness/lessons-log.md`** — append-only project history of root causes, decisions, and recurrences. **Search here when a problem feels familiar** or when deciding if something is a recurrence. Never edit old entries; use `/extract-lesson` to append new ones.

## Cognitive Honesty

- If unsure about an API, function, config, or fact, say "not sure". Never fabricate plausible-sounding answers.
- When user intent is ambiguous or the spec is missing key details, ask for clarification rather than guess. A two-second ask beats a two-hour wrong build.

## Computational Reliability

- All numerical computation must be executed by code. No mental math (arithmetic, date calculation, counting, unit conversion).

## Behavioral Discipline

- Read the surrounding code before editing. Match existing naming, error handling, and organization patterns — consistency with what's there usually beats "the right way" in the abstract.
- Never claim "fixed", "tests pass", or "works" unless a verification command was actually run and its output confirmed. Attach the evidence (command output, response body, or diff) to the claim.
- Before declaring a feature complete, enumerate the edge cases you considered (empty inputs, concurrency, failure paths, resource limits) and note which are handled vs deferred. "Happy path worked once" ≠ done.
- When fixing a bug, trace to root cause before applying a fix. Catch-and-ignore, suppressing warnings, or deleting failing assertions to "make it pass" is hiding, not fixing.
- Only do what was asked. No drive-by refactoring, no unrequested features, no "while I'm here" optimizations.

## Destructive Operations

- Before executing any destructive or irreversible operation (`rm -rf`, force push, DB migration, dropping data, external writes with side effects, removing dependencies), stop and present the plan for explicit user approval. Prior approval of similar operations does not extend to new ones.

## Architecture Ownership

- Architecture decisions — module boundaries, new abstractions, cross-cutting patterns, schema or API changes — belong to the user. When about to make one, surface the options with their tradeoffs; wait for direction. *"I'll put this in utils"* is an architecture decision.

## Rule Meta

- When adding a rule to this file, the bar is not "is it useful" but "is its value greater than the attention it dilutes from all existing rules".

<!-- ──────────────────────────────────────────────────────────────────────── -->
<!-- Project-specific rules below. Refer to .harness/architectural-intent.md  -->
<!-- Earn placement via recurrence — see .harness/lessons-log.md.             -->
<!-- ──────────────────────────────────────────────────────────────────────── -->

## Project-specific

- When adding a Route, include a sample payload in the route definition comment and link to the consumer's expected shape (repo URL or schema file). Delivery code must not assume any payload shape beyond what the consumer has documented. (_Promoted from lessons-log 2026-04-02 after recurrence of payload mismatch._)
