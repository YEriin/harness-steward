# Example — Minimal Web Service

A fictional project **pico-notifier** (a small Node.js service that accepts HTTP events and fans out to webhooks) with harness discipline applied.

This example shows **what sediment looks like in a real project** after running `bootstrap-new-project`, then a few weeks of development with occasional `extract-lesson` invocations. It is **not runnable code** — no `package.json`, no source. Only the `.harness/` + `CLAUDE.md` artifacts.

## Structure

```
examples/minimal-web-service/
├── README.md                          ← this file
├── CLAUDE.md                          ← minimal starting rules
└── .harness/
    ├── architectural-intent.md        ← filled-out Day-0 intent doc
    └── lessons-log.md                 ← 4 dated entries showing the flow
```

## What each file demonstrates

### `.harness/architectural-intent.md`
A filled-out version of `templates/architectural-intent.template.md` for a fictional system. Notice:
- **Section 2 (IS NOT)** does more work than Section 1 — defines specific boundaries
- **Section 3 (Key abstractions)** explicitly names *what each abstraction does NOT know about*
- **Section 4 (Evolution direction)** is direction-of-travel, not a roadmap

### `CLAUDE.md`
11 universal rules from the minimal template (across 6 rule sections + Project Context pointer), plus **one** project-specific rule that earned its place via recurrence (see entry 4 in lessons-log).

### `.harness/lessons-log.md`
Four entries showing the real flow:

1. **2026-03-01** — project bootstrapped (from `bootstrap-new-project`)
2. **2026-03-08** — structural defense added (category A from `extract-lesson`): PostToolUse hook to block `done` claims without test runs. Rule not added to `CLAUDE.md` because structural enforcement already catches it.
3. **2026-03-14** — first webhook payload schema mismatch logged (category E: one-off, logged only)
4. **2026-04-02** — second webhook schema mismatch logged; related to 2026-03-14. Now qualifies as recurrence → category C promoted to a `CLAUDE.md` rule.

This shows: **most entries stay as logs. The one promotion happened only after 2nd recurrence.** Exactly the flow the skills enforce.

## What to take from this example

- `.harness/` is small — two files, not a documentation portal
- Rules in `CLAUDE.md` stay lean — 12 total (11 universal template + 1 project-specific earned via recurrence). Well under `audit-harness`'s 15-rule dilution-risk threshold.
- Log entries are short and factual — no editorializing
- Dates matter — recurrence detection needs them

## Using this as a starter

Do **not** copy these files unchanged. The architectural-intent is for a fictional system. Run `bootstrap-new-project` in your own project instead, and use this example only as a visual reference for what your own sediment should look like after a few weeks.
