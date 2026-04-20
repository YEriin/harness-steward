# Architectural Intent

> _Project:_ **pico-notifier**
> _Last reviewed:_ 2026-04-02
>
> **Keep this document under 2 pages.** Edit quarterly at most.

---

## 1. What this system IS

A tiny HTTP service that accepts webhook events from internal services and fans them out to pre-configured outbound webhooks. Single-tenant, in-memory configuration, no persistence. Meant as glue between internal producers and external consumers.

---

## 2. What this system is NOT

- **Not a queue** — no durability guarantees; if pico-notifier restarts, in-flight events are lost. Callers must be OK with this.
- **Not a retry/backoff service** — we do exactly one outbound attempt per event. Retry is the consumer's problem or a separate upstream concern.
- **Not multi-tenant** — one deployment, one team's events. Routing config is baked in at deploy time.
- **Not a storage layer** — never writes events to disk, never queries a database.

---

## 3. Key abstractions

- **Event** — an incoming HTTP POST with a JSON payload and a routing key. Lives in `src/events/`. Does not know about outbound delivery, retries, or persistence.
- **Route** — a mapping from event routing-key to one or more outbound webhook URLs. Lives in `src/routing/`. Does not know about event payload contents (it's content-agnostic).
- **Delivery** — one-shot outbound HTTP request to a configured webhook. Lives in `src/delivery/`. Does not know about event semantics or what "success" means beyond HTTP 2xx.

---

## 4. Evolution direction

- We'll add structured logging for delivery success/failure, but still no retries.
- Routing config may move from env-baked to a read-only config file, but never to a database — this system stays stateless.
- We're explicitly rejecting any "buffering", "persistence", or "guaranteed delivery" features. If those become requirements, pico-notifier is not the right system — we'd build something else.
