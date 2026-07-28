---
name: thesis
trigger: Use when reasoning about LifeOS purpose, maturity level, the current→ideal hill-climb, or communicating what LifeOS is.
---

# Thesis — LifeOS Thesis → Hermes Operational Adaptation

## Purpose

This skill internalizes the LifeOS Thesis into the Hermes port. It is not a copy of the thesis — the thesis document (`LifeOS/install/LIFEOS/DOCUMENTATION/LifeOs/LifeOsThesis.md`) remains the source of truth. This skill is the operational adaptation that lets the Hermes-native DA reason and act according to the thesis.

## Three Layers in Hermes

The LifeOS three-layer model maps to Hermes-native equivalents:

| LifeOS Layer | Hermes Equivalent | What It Does |
|---|---|---|
| **DA** (primary interface) | Hermes desktop app | The chat surface, terminal, tools, and agent interaction |
| **Pulse** (dashboard) | Hermes desktop app + Freshness + cron | Status view, staleness grading, background observation |
| **LifeOS** (operating system) | Constitution + Hindsight + skills + cron | Doctrine, memory, capabilities, background services |

There is no separate Pulse web app. The Hermes desktop app IS the primary surface. LifeOS is the OS behind it.

## The Core Loop — Current State → Ideal State

Every turn is a hill-climb: pick the next move that reduces the gap between current and ideal state.

| State | Source | Hermes Mechanism |
|---|---|---|
| **Current state** | Hindsight recall, Conduit sensing, TELOS files, session context, LCM | `hindsight_recall`, Conduit `rollup.py`, TELOS Dropbox read, LCM context |
| **Ideal state** | TELOS (Dropbox), cognitive-graph goals, Hindsight `cat:telos` | TELOS files at `E:/Dropbox/ARON BIJL MSC/TELOS/`, cognitive-graph traversal, tagged recall |
| **Hill-climb** | The next move that reduces the gap | Every turn: observe, think, plan, build, execute, verify, learn |

This is already stated in `HERMES_CONSTITUTION.md` §1. The skill makes it actionable: before acting, recall current state; after acting, check whether the gap narrowed.

## LifeOS Maturity Model (LifeOS-MM)

| Level | Name | Description | Hermes Port Position |
|---|---|---|---|
| **C1** | Chat | Stateless single-turn exchange | — |
| **C2** | Memory | Cross-turn context retention | — |
| **C3** | Skills | Reusable procedural capabilities | — |
| **A1** | Persona | Persistent identity and voice | — |
| **A2** | Autonomous | Persistent, memory-rich, goal-aware | **Hermes + LifeOS port operates here** |
| **A3** | Integrated | Full integration across every system the principal uses | **Target** |
| **AS1** | Spark-aware | Captures and preserves human sparks | — |
| **AS2** | Spark-driven | Actively uses sparks to shape work | **Port targets this** |
| **AS3** | Day-in-the-life | Believable 2036 day-in-the-life | **Aspirational target** |

The Hermes + LifeOS port is an AS2 capability. The port itself — persistent memory via Hindsight, goal-awareness via TELOS, skill-based capabilities — is what moves the system from A1 to A2. AS3 requires integration across every system the principal uses, which is the long-term target.

## Lineage

LifeOS traces to *The Real Internet of Things* (2016), which envisioned the DA as the primary interface between a person and their digital life. The DA-as-primary-interface vision is realized in Hermes: the desktop app is where the principal interacts with their system, and LifeOS is the OS doctrine behind it. This is context for understanding *why* LifeOS exists, not operational instruction.

## Pulse → Hermes Surface Mapping

| Pulse Module | Hermes Equivalent |
|---|---|
| Observability | Hermes LCM + session logs + terminal |
| Voice | Hermes TTS plugin |
| iMessage / Telegram | Hermes gateway integrations |
| Cron / Assistant / Worker | Hermes cron system |
| Current State vs Ideal State | Freshness `check.py` + TELOS recall + Conduit rollup |
| Goals & Workflows | Hindsight `cat:telos` + ISA tracking |
| Day-in-the-Life | Not yet implemented; future capability |
| Respark signals | Not yet implemented; future capability |

## Respark — The Human Reclamation Layer

Respark is the process of reclaiming human attention from digital noise:

| Concept | Meaning | Hermes Destination |
|---|---|---|
| **Sparks** | Things that genuinely energize the principal | TELOS (Dropbox) + Hindsight `cat:telos` with `domain:respark` |
| **Play** | Space for unstructured exploration | Preserved in TELOS; DA asks about spark-related context |
| **Integration** | Weaving sparks into daily work | DA surfaces spark-relevant opportunities during planning |

The DA should ask about and preserve spark-related information during interviews and conversation. Sparks are durable identity data — they belong in Hindsight with `cat:telos` and `domain:respark` tags.

## 2036 Reverse-Engineering Heuristic

Use as a decision quality gate: **Does this move us toward a believable 2036 day-in-the-life?**

When evaluating a feature, capability, or port decision:
1. Would this exist in a mature, AS3 system?
2. Does it reduce friction between the principal and their goals?
3. Does it preserve or reclaim human attention?
4. Is it a stepping stone or a dead end?

If the answer is no or unclear, the feature is premature. This heuristic applies to the port itself — every skill, cron job, and integration should pass this gate.

## Naming Conventions

| Don't Say | Say Instead |
|---|---|
| "LifeOS is scaffolding for AI" | "LifeOS is the Life Operating System" |
| "LifeOS is a dashboard" | "The Hermes desktop app is the primary surface; LifeOS is the OS behind it" |
| A hardcoded DA name | "Your DA" in public-facing content; "HAL" in this deployment (from SOUL.md) |
| "Pulse shows…" | "The status view shows…" or "Freshness reports…" |

Always anchor to the maturity model (AS3 target) and the 2036 reverse-engineering target when describing what LifeOS is or why a capability exists.

## Cross-references

- **`HERMES_CONSTITUTION.md`** — §1 (Operating aim), §2 (Identity), §3 (Execution loop)
- **Memory skill** — Hindsight as canonical memory with mutation tier mapping
- **Schema skill** — USER/ directory → Hermes-native destination mapping
- **Conduit skill** — Current-state sensing via Windows polling
- **Freshness skill** — A-F staleness grading for TELOS and identity files
- **`LifeOS/install/LIFEOS/DOCUMENTATION/LifeOs/LifeOsThesis.md`** — Source of truth (do not modify)
