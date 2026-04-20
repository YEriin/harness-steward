# Architectural Intent

> _Project:_ **`<FILL IN: project name>`**
> _Last reviewed:_ `<FILL IN: YYYY-MM-DD>`
>
> **Keep this document under 2 pages.** Long is worse than short. Edit quarterly at most — see `references/writing-architectural-intent.md`.

---

## 1. What this system IS

<!--
2-3 sentences. The central purpose, as you'd explain to a new engineer on Day 1.
State what IS, not what you hope.

Example:
"A CLI tool that ingests structured logs from cloud services, normalizes
them into a single query-friendly format, and writes to a single analytics
database. Single-tenant; one deployment per customer."
-->

`<FILL IN>`

---

## 2. What this system is NOT

<!--
Hard boundaries. Each bullet is a "we don't do X" that prevents scope creep.
The negatives do more work than the positives.

Examples:
- Not a real-time system — eventual consistency is fine
- Not multi-tenant — one instance, one customer's data
- Not a data warehouse — ephemeral processing only
-->

- `<FILL IN>`
- `<FILL IN>`
- `<FILL IN>`

---

## 3. Key abstractions

<!--
3-5 core concepts. For each:
- Name
- One-sentence purpose
- Which module owns it
- What it's allowed to know about / NOT allowed to know about

The "does not know about" half defines the coupling you refuse to allow.

Example:
- **User** — authenticated identity. Lives in `auth/`. Does not know about billing.
- **Session** — user's current interaction context. Lives in `session/`. Does not know about persistence backend.
- **Job** — async task unit. Lives in `jobs/`. Does not know about HTTP / responses.
-->

- **`<FILL IN: name>`** — `<FILL IN: purpose>`. Lives in `<FILL IN: module>`. Does not know about `<FILL IN: forbidden coupling>`.
- **`<FILL IN>`** — `<FILL IN>`. Lives in `<FILL IN>`. Does not know about `<FILL IN>`.
- **`<FILL IN>`** — `<FILL IN>`. Lives in `<FILL IN>`. Does not know about `<FILL IN>`.

---

## 4. Evolution direction

<!--
Where this is heading in 3-6 months. Direction of travel, NOT a roadmap.

Examples:
- We'll add streaming responses but not streaming ingestion.
- Auth will split into a separate service; code should anticipate this.
- We won't support on-prem deploys; cloud-only.
-->

- `<FILL IN>`
- `<FILL IN>`
