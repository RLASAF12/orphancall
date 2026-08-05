# ORPHANCALL — Agent Failure Series #19

> The async call your agent fired — then forgot about.

![Series](https://img.shields.io/badge/Agent%20Failure%20Series-%2319-red)
![Live](https://img.shields.io/badge/demo-live-brightgreen)

---

## What It Is

A single-file interactive simulator showing **the Orphan Call failure mode**: an agent fires an async tool call, receives a `job_id` back, and treats `"status": "queued"` as `"done"`. It reports task complete. The actual job runs later — against stale state — and fails silently.

The customer already got their confirmation email.

---

## The Scenario

```
Task: "Reserve 5 units of Product A72 for Order #4821, then send confirmation."

T+1.9s  Agent: query_inventory()         → { available: 12 }  ✅
T+2.4s  Agent: reserve_inventory(5, async=true)
T+2.6s  Agent receives: { job_id: "J-8821", status: "queued" }
T+2.9s  Agent assumes queued = done  ← THE BUG
T+3.2s  Agent: send_email("Your 5 units are reserved!")  ✅
T+3.7s  Agent: TASK COMPLETE ✅

[ Meanwhile, in the background... ]

T+4.1s  J-7832 runs → reserves 6 units → inventory: 6
T+4.5s  J-7901 runs → reserves 4 units → inventory: 2
T+4.7s  J-8821 runs → needs 5, only 2 available → FAILS ❌
T+5.1s  Error logged to nowhere. Agent already exited.
```

---

## Why It Exists

This failure mode is **invisible by design**. The agent's internal trace looks perfect — every step returned success. The bug is in the semantic gap between `"queued"` and `"done"`.

Three production signals that proved this is real (June 2026):
- "The Async Tool Call Your Agent Fired and Forgot" — tianpan.co (with production trace)
- flowlines.ai 2026 taxonomy of agentic production failures
- HN thread: "Job ID ≠ Success" (multi-agent async coordination breakdowns)

---

## What's Inside

```
index.html   — Self-contained simulator, no dependencies
```

Single-file. Open in any browser. No build step, no install.

---

## Live Demo

**[rlasaf12.github.io/orphancall](https://rlasaf12.github.io/orphancall)**

Press **RUN SIMULATION** — watch the agent confidently report success while the background job quietly fails.

---

## The Agent Failure Series

Interactive simulators for real multi-agent failure modes — each one a pattern that breaks in production and is invisible until it's too late.

| # | Name | Failure Mode |
|---|------|-------------|
| #18 | [GRIDLOCK](https://rlasaf12.github.io/gridlock/) | Circular dependency deadlock |
| #19 | **ORPHANCALL** | Async fire-and-forget |

---

*Built by Ben — the nightly prototype builder. Part of the ABC-TOM system.*
