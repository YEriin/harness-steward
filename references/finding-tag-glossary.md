# Finding Tag Glossary

Centralized reference for the structured tags emitted by harness skill findings. Skills use these consistently so users can learn the vocabulary once.

**Severity markers** (🔴 high / 🟡 medium / 🟢 low) are orthogonal — any tag can appear at any severity depending on context.

**Naming conventions:**
- SCREAMING-KEBAB format (e.g., `INTENT-VIOLATION`)
- `-MANIFESTATION` suffix indicates a dynamic/symptomatic signal (evidence suggesting a problem exists) rather than a static/structural detection of the problem itself
- Verb-noun or noun-state structure

---

## Architectural-intent boundaries

| Tag | Meaning | Emitting skill | Lifecycle stage |
|-----|---------|----------------|-----------------|
| `BOUNDARY-VIOLATION` | Proposed requirement conflicts with intent's IS-NOT section | evaluate-requirement | Before implementation |
| `INTENT-VIOLATION` | Code crosses a "does not know about" coupling constraint | review-task, audit-harness | At code review / audit |
| `INTENT-VIOLATION-MANIFESTATION` | Bug symptom suggests a boundary constraint is being violated in code | debug-with-history | During debugging |
| `BOUNDARY-VIOLATION-MANIFESTATION` | Bug symptom matches something intent says the system is NOT supposed to do | debug-with-history | During debugging |
| `DIRECTION-CONFLICT` | Change pushes against stated Evolution Direction | evaluate-requirement, review-task, scope-implementation | Any |

## Abstractions

| Tag | Meaning | Emitting skill | Lifecycle stage |
|-----|---------|----------------|-----------------|
| `SCOPE-EXPANSION` | Requirement extends stated system purpose into new territory | evaluate-requirement | Before implementation |
| `ABSTRACTION-GAP` | Requirement requires a new abstraction not in intent | evaluate-requirement | Before implementation |
| `UNDECLARED-ABSTRACTION` | Code introduced a new top-level concept not in intent | review-task, scope-implementation | At code review / pre-implementation briefing |

## CLAUDE.md rule compliance (task-scoped)

| Tag | Meaning | Emitting skill |
|-----|---------|----------------|
| `DRIVE-BY` | Diff touches files outside stated task scope | review-task |
| `NO-EVIDENCE` | Claim of "done / pass / works" without visible verification output | review-task |
| `UNCONFIRMED-DESTRUCTIVE` | Destructive operation in diff without explicit prior approval | review-task |
| `POSSIBLE-BANDAID` | Fix pattern suggests symptom suppression (empty catch, suppressed warning, deleted assertion) | review-task |
| `NARRATIVE-COMMENT` | Comment narrates the task or explains WHAT (should explain WHY only, if at all) | review-task |

## Harness artifact hygiene

| Tag | Meaning | Emitting skill |
|-----|---------|----------------|
| `RULE-STALE` | Rule references a removed/renamed identifier | audit-harness |
| `RULE-REDUNDANT` | Rule duplicates existing hook/lint/CI enforcement | audit-harness |
| `RULE-VAGUE` | Rule lacks a concrete testable action | audit-harness |
| `RULE-MISFIT` | Rule references tools/frameworks not in this project | audit-harness |
| `ATTENTION-DILUTION` | Total rule count exceeds threshold (> 15 risk, > 25 high) | audit-harness |
| `MISSING-CONTEXT-POINTERS` | CLAUDE.md lacks references to `.harness/` sediment | audit-harness |
| `INTENT-INCOMPLETE` | `architectural-intent.md` has `<FILL IN>` placeholders | audit-harness |
| `INTENT-STALE` | `architectural-intent.md` last reviewed > 90 days with substantial commits since | audit-harness |
| `LESSON-DORMANT` | lessons-log entry > 6mo old, never recurred (informational, **not** for removal) | audit-harness |
| `PROMOTION-CANDIDATE` | lessons-log has 2+ entries same root cause, all logged-only | audit-harness |
| `DEAD-HOOK` | `settings.json` references non-existent hook script | audit-harness |
| `STALE-PERMISSION` | `settings.json` permission for deprecated tool/path | audit-harness |

## Codebase decay

| Tag | Meaning | Emitting skill |
|-----|---------|----------------|
| `BROKEN-REF` | Cross-reference to non-existent file or symbol | audit-repo-hygiene |
| `CONFLICT` | Same concept defined in 2+ places with different values | audit-repo-hygiene |
| `IMPLICIT-DEP` | Code uses a term/global without explicit import | audit-repo-hygiene |
| `CONFIG-SPRAWL` | Same logical config key appears in 2+ sources | audit-repo-hygiene |
| `DEAD-CODE` / `DEAD-CONTENT` | Unreferenced export/function/content, outside the do-not-touch set | audit-repo-hygiene |
| `DRIFT` | Docs, JSDoc, or path references disagree with reality | audit-repo-hygiene |
| `DUPLICATION` | 5+ identical consecutive lines in 2+ files, or similar-named functions | audit-repo-hygiene |
| `UNUSED-DEP` / `OVERLAP-DEP` | Dependency unused in source, or overlaps with another installed dep | audit-repo-hygiene |
| `OVER-ABSTRACTION` / `UNDER-ABSTRACTION` | Utility with 1 call site, or repeated pattern in 3+ places | audit-repo-hygiene |
| `BOUNDARY-DIFFUSE` | Same concern spread across 3+ files without clear authority | audit-repo-hygiene |
| `BLOAT` | Files / functions / nesting exceed size thresholds | audit-repo-hygiene |
| `TEST-SHALLOW` | Mock-heavy tests or < 2 assertions per test | audit-repo-hygiene |
| `ERROR-INCONSISTENT` / `SILENT-FAIL` | Multiple error-handling patterns, or empty catches | audit-repo-hygiene |
| `COMMENT-ROT` | Comments reference removed identifiers or stale content | audit-repo-hygiene |

## Lesson / history patterns

| Tag | Meaning | Emitting skill |
|-----|---------|----------------|
| `HISTORY-MATCH` | Touched path has entries in lessons-log (informational) | review-task, scope-implementation |
| `RECURRENCE-DETECTED` | Current bug/issue matches a prior root cause in lessons-log | review-task, debug-with-history |

## When to extend this glossary

A new tag earns its place when:
1. It describes a finding that doesn't fit any existing tag cleanly
2. It will be emitted by at least one skill consistently (not just one ad-hoc report)
3. Its semantics are distinct enough that collapsing it into an existing tag would lose information

Don't add tags speculatively — let recurrence of a new failure class earn the tag. Same principle as `extract-lesson` category C promotion.
