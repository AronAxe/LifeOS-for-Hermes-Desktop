---
name: memory
trigger: Use when managing durable memory, curation boundaries, or mapping LifeOS memory concepts to Hermes Hindsight.
---

# Memory — LifeOS File-Memory → Hermes Hindsight

## Purpose

This skill replaces the LifeOS file-system-based memory runtime (`~/.claude/LIFEOS/MEMORY/`, `MutationTier.ts`, `MemoryReviewer`, `MemoryGraph.ts`, and all TypeScript memory tooling) with **Hindsight** as the canonical associative-memory layer. There is no file-tree memory in Hermes. Hindsight provides `recall`, `retain`, and `reflect` operations that map directly to the LifeOS memory responsibilities. The TypeScript tools are not ported as runtime code; their *responsibilities* are mapped to Hermes-native equivalents.

## Mutation Tier Mapping

The LifeOS four-tier mutation system maps to Hermes layer boundaries:

| LifeOS Tier | LifeOS Behavior | Hermes Equivalent | Hindsight Tags | document_id Pattern |
|---|---|---|---|---|
| **A** (auto set-overwrite) | `PRINCIPAL_MEMORY.md`, `DA_MEMORY.md` — direct overwrite | `hindsight_retain` with set-replace semantics | `cat:identity`, `cat:telos` | `user:{id}:identity:principal`, `user:{id}:config:operational_rules` |
| **B** (logged append) | `PROJECTS.md`, `CONTACTS.md`, `KNOWLEDGE/`, `IDEAS/` — append with log | `hindsight_retain` with unique document_id per item | `cat:entity`, `cat:knowledge` | `user:{id}:project:{slug}`, `user:{id}:entity:person:{slug}`, unique per item |
| **C** (propose-only) | identity, style, definition, resume, operational-rules — require approval | Cognitive-graph capture + `hindsight_retain` after explicit user confirmation | `cat:identity`, `durability:core` | `user:{id}:identity:principal`, `user:{id}:wisdom:{domain}` |
| **D** (untouchable) | Credentials, config, code — never auto-modified | Files outside Hermes memory scope — never auto-retain | — | — |

## Curation Coverage

| LifeOS Curated File | Hermes Destination | Trigger | Cadence |
|---|---|---|---|
| `PRINCIPAL_MEMORY.md` | Hindsight `cat:identity` + `cat:telos` | Explicit user fact or TELOS update | On change |
| `DA_MEMORY.md` | SOUL.md + Hindsight `cat:identity` | Agent identity update | On change |
| `PROJECTS.md` | Hindsight `cat:knowledge` + `project:` tag | Project creation/update | On change |
| `CONTACTS.md` | Hindsight `cat:entity` | New contact / update | On change |
| `KNOWLEDGE/` | Hindsight `cat:knowledge` | Learning event | On change |
| `IDEAS/` | Hindsight `cat:knowledge` + Amber routing | Amber capture | On capture |
| `LEARNING/` | Hindsight `cat:learning` | Session completion, failure | On completion |
| `WISDOM/` | Hindsight `cat:wisdom` | `hindsight_reflect` synthesis | Cron `lifeos-wisdom-synthesis` every 6h |
| `RELATIONSHIP/` | Hindsight `cat:identity` | Relationship context update | On change |
| TELOS summary | Hindsight `cat:telos` with `source:dropbox_telos` | TELOS file change | On TELOS refresh |
| ISA sync | Workspace/ISA files (NOT Hindsight) | Phase transition | Per-ISA |
| Security | Hermes security logs (NOT Hindsight) | Event-driven | Continuous |
| Observability | Hermes LCM + session logs (NOT Hindsight) | Built-in | Continuous |
| TELOS source files | Dropbox `E:/Dropbox/ARON BIJL MSC/TELOS/` | Manual / cron refresh | As needed |
| Credentials | Never touched | — | — |
| Code | Git repository | — | — |

## Memory Lifecycle

The LifeOS MemoryReviewer cadence (8 turns / 30 min / 2 idle → reviewer subprocess) is replaced by the Hermes turn lifecycle:

| Phase | LifeOS | Hermes | Mechanism |
|---|---|---|---|
| **Turn start** | `LoadMemory` + `MemoryDeltaSurface` | Recall relevant context | `MemoryManager.prefetch_all()` → `hindsight_recall` → inject into system prompt |
| **Turn end** | `MemoryReviewFire` | Retain durable facts from the turn | `MemoryManager.sync_all()` → `hindsight_retain` with rich conversation content |
| **Background** | `MemoryReviewer` subprocess | Synthesize patterns asynchronously | `hindsight_reflect` via cron `lifeos-wisdom-synthesis` (every 6h) |
| **Session switch** | Manual flush | Flush old buffer, mint fresh document_id | Hindsight plugin session-switch hook (`/reset`, `/new`, `/resume`) |

Key difference: the cadence is built into Hermes' turn lifecycle, not a separate subprocess. The `🧠 MEMORY` delta surface is replaced by Hermes per-turn context injection (already handled by LCM + Hindsight plugin).

**Critical:** Pass the richest useful conversation content to `retain`. Do not pre-summarize or pre-distill sessions before retaining. Hindsight extracts facts itself; raw content is not stored verbatim as memory.

## Proposal Subtypes

LifeOS proposal subtypes map to Hermes cognitive-graph and Hindsight destinations:

| LifeOS Proposal Kind | Hermes Destination | Confirmation Required | Hindsight Tags |
|---|---|---|---|
| `identity` | Cognitive-graph `value`/`identity` node + Hindsight | Yes — explicit user confirmation | `cat:identity`, `durability:core` |
| `style` | SOUL.md update + Hindsight | Yes — SOUL.md is identity | `cat:identity` |
| `definition` | Hindsight with stable document_id | Yes | `cat:knowledge` |
| `canonical-content` | Hindsight | Yes | `cat:knowledge` |
| `resume` | Hindsight | Yes | `cat:identity`, `durability:core` |
| `operational-rule` | SOUL.md or constitution + Hindsight | Yes | `cat:identity` |
| `projects` | Hindsight with `project:` tag | No (Tier B) | `cat:knowledge` |
| `contacts` | Hindsight with entity document_id | No (Tier B) | `cat:entity` |

## Directory Inventory Mapping

| LifeOS `MEMORY/` Subdir | Hermes Equivalent | What Lives There |
|---|---|---|
| `KNOWLEDGE/` | Hindsight `cat:knowledge` | Distilled ideas, research, architectural decisions |
| `WORK/` | Hermes workspace / ISA files (NOT memory) | Active work state and evidence |
| `LEARNING/` | Hindsight `cat:learning` | Session learnings, failure postmortems, fixes |
| `WISDOM/` | Hindsight `cat:wisdom` + cron `lifeos-wisdom-synthesis` | Domain frames, cross-cutting principles, mental models |
| `RELATIONSHIP/` | Hindsight `cat:identity` | Relationship context and history |
| `OBSERVABILITY/` | Hermes LCM + session logs (NOT Hindsight) | Context receipts, compression, transcript continuity |
| `SECURITY/` | Hermes security logs (NOT Hindsight) | Security event logs |
| `STATE/` | Hermes session/workspace state (NOT Hindsight) | Live session state, work registries |
| `VOICE/` | Hermes TTS plugin logs (NOT Hindsight) | TTS output logs |

## What NOT to Put in Hindsight

These belong in Hermes session, workspace, or logging layers — **never** in Hindsight:

- **Active task state** — ISA checklists, phase state, `work.json`-style live state
- **Pending approval/proposal queues** — Hermes orchestration layer manages these
- **Tool telemetry** — tool activity logs, cost logs, security JSONL streams
- **High-frequency observability events** — event firehoses that would flood memory
- **Pre-summarized session content** — pass rich content to `retain`; let Hindsight extract

## Cross-references

- **`HERMES_CONSTITUTION.md` §5** — Memory boundaries (canonical statement)
- **`PORT_SCHEMAS/hindsight_memory_schema.md`** — Full Hindsight bank layout, tag taxonomy, document_id strategy, operation triggers
- **`PORT_SCHEMAS/hook_mapping.md`** — MemoryTurnStart, MemoryReviewFire, MemoryHealthGate, WorkCompletionLearning hook mappings
- **Freshness skill** — A-F staleness grading for TELOS and identity files
- **Amber skill** — Idea capture and preservation loop (routes ideas into Hindsight)
- **Conduit skill** — Current-state sensing (feeds daily record into Hindsight)

## Configuration Notes

- `memory_enabled: false` in `config.yaml` is **intentional**. Enabling it activates Hermes' built-in bolt-on memory system, which is NOT used. Hindsight runs independently via its own plugin + `hindsight_recall`/`retain`/`reflect` tools + the cron job.
- TELOS is loaded from `E:/Dropbox/ARON BIJL MSC/TELOS/` and retained with `document_id: "user:aron:telos"`, tags `["cat:telos", "cat:identity", "durability:core", "source:dropbox_telos"]`.
- Cron `lifeos-wisdom-synthesis` (every 6h, `deliver: local`): calls `hindsight_reflect` to synthesize patterns, optionally retains output with `cat:wisdom` and `document_id: "user:aron:wisdom:synthesized"`.
