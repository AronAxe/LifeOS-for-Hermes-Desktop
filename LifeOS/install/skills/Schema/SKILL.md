---
name: schema
trigger: Use when organizing personal information, mapping LifeOS USER/ files to Hermes destinations, or understanding the schema-to-Hermes mapping.
---

# Schema — LifeOS USER/ Directory → Hermes-Native Mapping

## Purpose

This skill adapts the LifeOS Schema (the `USER/` directory shape) to Hermes-native destinations. The LifeOS `USER/` file tree is a biography-style organization where everything the DA knows about the principal lives. In Hermes, that tree is **reference scaffolding**, not the runtime source of truth. This skill documents where each kind of personal information actually lives in the Hermes port.

## The One Rule (Hermes Version)

| LifeOS | Hermes |
|---|---|
| Single concept → single file at `USER/` root | Single concept → Hindsight document with stable `document_id` |
| Multi-file → directory with `README.md` | Multi-faceted → cognitive-graph node with typed edges |

The biography-style organization is preserved through Hindsight tags + cognitive-graph relationships, not a file tree.

## Category → Hindsight Tag Mapping

| LifeOS Category | Hermes Destination | Hindsight Tags |
|---|---|---|
| `identity` | Hindsight + SOUL.md | `cat:identity`, `durability:core` |
| `voice` | SOUL.md | `cat:identity` |
| `mind` | Hindsight + cognitive-graph | `cat:wisdom`, `cat:knowledge` |
| `taste` | Hindsight | `cat:knowledge` |
| `shape` | Hindsight + TELOS | `cat:telos` |
| `ops` | Hermes config + Hindsight | `cat:knowledge` |
| `domain` | Hindsight + TELOS + cognitive-graph | `domain:`-specific tags |

## Kind → Hermes Render Equivalent

| LifeOS Kind | Hermes Render Equivalent |
|---|---|
| `collection` | `hindsight_recall` returns list of items under a tag |
| `narrative` | `hindsight_recall` returns a coherent narrative document |
| `reference` | `hindsight_recall` returns key/value pairs (cognitive-graph nodes) |
| `index` | `hindsight_recall` returns a directory listing (cognitive-graph traversal) |

## Frontmatter Contract → Hermes Metadata

| LifeOS Frontmatter | Hermes Equivalent |
|---|---|
| `category` | Hindsight `cat:` tag |
| `kind` | Hermes render equivalent (see above) |
| `publish: false` | Private (default in Hindsight) |
| `publish: daemon` | **Removed** — no daemon broadcast in Hermes |
| `review_cadence` | Freshness skill (`check.py` with A-F grading) |
| `interview_phase` | Interview skill (already ported) |
| `last_updated` | Hindsight document metadata (automatic) |

## Pulse → Hermes Surface

LifeOS Pulse is a web-based React dashboard. Hermes has no Pulse web app. The Hermes desktop app itself is the primary surface.

| Pulse Route | Hermes Equivalent |
|---|---|
| `/life` | `hindsight_recall` with `cat:identity` or `cat:telos` |
| `/life/c/:category` | `hindsight_recall` with category-specific `cat:` tag |
| `/life/d/:domain` | `hindsight_recall` with `domain:` tag |
| `/life/g/:goal` | `hindsight_recall` with `cat:telos` + goal context |
| `/freshness` | `python LifeOS/install/skills/Freshness/Tools/check.py` (text/JSON) |
| `/observability` | Hermes LCM status, session logs, terminal |
| `/voice` | Hermes TTS plugin |
| `/cron` | Hermes cron system (`hermes cron list`) |

## File Inventory Mapping

| LifeOS `USER/` File | Hermes Destination | Notes |
|---|---|---|
| `PRINCIPAL/PRINCIPAL_IDENTITY.md` | Hindsight `cat:identity` + `cat:telos` | Canonical source: configured TELOS source in `LifeOS/install/HERMES.md` |
| `PRINCIPAL/PRINCIPAL_MEMORY.md` | Hindsight `cat:identity` | Stable `document_id: user:{id}:identity:principal` |
| `DIGITAL_ASSISTANT/DA_IDENTITY.md` | `$HERMES_HOME/SOUL.md` | Agent identity (HAL) |
| `DIGITAL_ASSISTANT/DA_MEMORY.md` | SOUL.md + Hindsight `cat:identity` | Agent operational memory |
| `PROJECTS.md` | Hindsight `cat:knowledge` + `project:` tag | Per-project document_ids |
| `CONTACTS.md` | Hindsight `cat:entity` | `user:{id}:entity:person:{slug}` |
| `KNOWLEDGE/` | Hindsight `cat:knowledge` | Distilled knowledge |
| `LEARNING/` | Hindsight `cat:learning` | Session learnings and postmortems |
| `WISDOM/` | Hindsight `cat:wisdom` | Synthesized via cron |
| `RELATIONSHIP/` | Hindsight `cat:identity` | Relationship context |
| `VOICE/` | Hermes TTS plugin | Not memory |
| `OPS/` | Hermes config + Hindsight `cat:knowledge` | Operational rules |
| `RESUME/` | Hindsight `cat:identity` + `durability:core` | After explicit confirmation |

## Removed Components

| Component | Reason |
|---|---|
| Daemon broadcast system | macOS-specific `launchd` service; not applicable to Hermes |
| Pulse React components | No Pulse web dashboard; Hermes desktop app is the surface |
| `fs.watch` indexer | Hindsight provides recall; no filesystem watcher needed |
| `publish: daemon` frontmatter | No daemon to broadcast to |

## Cross-references

- **Interview skill** — Principal state discovery and TELOS extraction (already ported)
- **Freshness skill** — A-F staleness grading for TELOS and identity files
- **Memory skill** — Hindsight as canonical memory layer with mutation tier mapping
- **`HERMES_CONSTITUTION.md`** — §2 (Identity), §5 (Memory boundaries), §6 (Layer boundaries)
- **`PORT_SCHEMAS/hindsight_memory_schema.md`** — Full Hindsight tag taxonomy and document_id strategy

## Key Principle

The `USER/` file tree in the LifeOS repository is **reference scaffolding** for understanding the schema concept. It is not the runtime source of truth. In Hermes:

- **TELOS** lives at the configured TELOS source in `LifeOS/install/HERMES.md`
- **Agent identity** lives in `$HERMES_HOME/SOUL.md`
- **Durable memory** lives in Hindsight
- **Cognitive graph** holds typed decision architecture
- **Workspace/ISA files** hold active work state
