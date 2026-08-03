---
name: Synapse
version: 1.0.0
description: "The weighted input router (supersedes Amber). Central model: Conduit and Feed sense; Synapse routes; Cortex stores. Captures ideas and signals, preserves them in Hindsight before grading, grades against TELOS, and routes to destinations. USE WHEN: capture idea, process inputs, route information, save for later."
effort: medium
---

# Synapse — Weighted Input Router

## What It Does

Synapse is the weighted input router of the LifeOS ecosystem, superseding the legacy **Amber** router (though the "amber ledger" remains the preservation concept). In the central model: **Conduit** and **Feed** sense the world; **Synapse** routes the inputs; and **Cortex** stores them.

The core flow is: **Capture → Journal → Grade → Route → Resurface**.

Crucially, **preservation happens before grading**. Weak signals are not discarded; they are caught in the "amber ledger", meaning they are safely retained in Hindsight immediately, unconditionally, before any grading or routing logic applies.

## The Capture Contract

Every captured input must preserve the following fields to ensure proper provenance and routing.

| Field | Required | Meaning |
|-------|----------|---------|
| `source` | yes | The origin of the input (e.g., manual, feed, lifelog) |
| `external_id` | yes | The input's original ID (e.g., tweet id, URL hash) |
| `url` / `content` | yes | The normalized URL or raw text if no URL exists |
| `captured_at` | yes | ISO-8601 timestamp of when it entered Synapse |
| `content_kind` | yes | E.g., article, video, tweet, note, tool |
| `title` / `author`| no | Metadata if provided by the input source |
| `privacy_class`| yes | `public` or `personal` to gate cloud/external sync |

## Capture Invariants

These properties are non-negotiable:

- **Write-ahead:** retain the capture before any grade or route decision. If downstream work fails, the signal remains recoverable.
- **Idempotent:** derive `capture_id` from normalized URL/content identity, falling back to `source` + `external_id`; retries must not create duplicate durable records.
- **Async downstream:** grading and routing may follow capture; they must not make preservation conditional on model availability.
- **Privacy-gated:** `privacy_class: personal` cannot cross a local-to-cloud or shared destination without an explicit permitted route.
- **Rich source:** retain the rawest useful URL, text, transcript, or note together with provenance; do not pre-distill away the evidence.

## Hermes-Native Mapping

Synapse moves away from a custom D1/JSONL runtime and maps its operations directly to Hermes primitives:

- **Amber Ledger (Preservation)**: Mapped to `hindsight_retain` with a stable per-user `document_id` such as `user:{user_id}:synapse:{capture_id}` and provenance tags (e.g., `cat:synapse`, `source:amber_capture`). The raw capture is preserved immediately; never hardcode a real person, account, or private path in a public skill.
- **Grade**: Mapped to `hindsight_recall` and `hindsight_reflect` combined with DA judgment and TELOS alignment.
- **Route**: Synapse routes graded inputs to Hermes-native destinations:
  - Hindsight categories (e.g., `cat:knowledge`, `cat:learning`)
  - Cognitive graph (values, models)
  - Workspace / ISA files (for active tasks)
  - Approved external destinations

## Memory Boundaries

Synapse strictly respects Hermes memory boundaries:
- **Do not** put active task state, checklists, or work registries into Hindsight.
- **Do not** put tool telemetry, raw JSONL event logs, or approval queues into Hindsight.
- Hindsight is reserved for durable facts, entities, and routed knowledge.

## Boundaries with Other Subsystems

- **Conduit & Feed**: These are the sensory organs. They gather data; Synapse processes and routes it.
- **Cortex**: The durable store (Hindsight) where Synapse sends routed knowledge.
- **Harvest**: Processes long-term synthesis, whereas Synapse handles immediate triage and routing of new inputs.

## Examples

### Example 1 — Capturing a New Idea
`/skill Synapse "capture https://example.com/insight"`
1. Normalizes the URL and generates a stable capture ID.
2. Immediately preserves it via `hindsight_retain` with `user:{user_id}:synapse:{capture_id}`, `cat:synapse`, `source:amber_capture`, and the capture contract fields.

### Example 2 — Grading and Routing
`/skill Synapse "route unrouted captures"`
1. Recalls unrouted captures from Hindsight.
2. Uses `hindsight_recall` to fetch TELOS context.
3. Grades the captures against TELOS (DA judgment).
4. Routes passing captures to the Cognitive graph or Hindsight (`cat:knowledge`), updating the original record with a `routed:true` tag.
