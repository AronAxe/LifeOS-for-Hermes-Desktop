---
name: pulse
trigger: Use when reasoning about the Pulse dashboard system, DA subsystem architecture, terminal tab state, metadata surfaces, or mapping Pulse modules to Hermes-native equivalents.
---

# Pulse — LifeOS Dashboard → Hermes-Native Mapping

## Purpose

Pulse is the Life Dashboard — the visible surface of LifeOS. In Hermes, there is no separate Pulse web app. The Hermes desktop app is the primary surface. This skill maps every Pulse subsystem, module, and surface to its Hermes-native equivalent, so the DA can reason about "what Pulse did" in Hermes terms.

## Pulse Daemon → Hermes Runtime

The unified Pulse daemon (`~/.claude/LIFEOS/PULSE/`, port 31337, `com.lifeos.pulse`) was a single always-on macOS process. In Hermes, its responsibilities are distributed across the native runtime:

| Pulse Module | LifeOS Function | Hermes Equivalent | Notes |
|---|---|---|---|
| **Cron** (`pulse.ts`) | Scheduled jobs | Hermes cron system | `hermes cron list`, `cronjob` tool. No launchd. |
| **Voice** (`VoiceServer/voice.ts`) | ElevenLabs TTS notifications | Hermes TTS plugin | `text_to_speech` tool. No `/notify` endpoint. |
| **Hooks** (`modules/hooks.ts`) | Skill-guard and agent-guard validation | Hermes tool approval + safety middleware | Built into Hermes runtime. No hook scripts. |
| **Observability** (`Observability/observability.ts`) | Data APIs + dashboard | Hermes LCM + session logs + terminal | `lcm_status`, `lcm_inspect`, session_search. No JSONL pipeline. |
| **Telegram** (`modules/telegram.ts`) | grammY polling bot | Hermes gateway integrations | Hermes gateway handles Telegram natively. |
| **iMessage** (`modules/imessage.ts`) | SQLite polling bot | Hermes gateway integrations | iMessage not available on Windows. |
| **Worker** (`checks/github-work.ts`) | GitHub Issues work polling | Hermes cron + `gh` CLI | Cron job that runs `gh issue list` periodically. |
| **Assistant** (`Assistant/module.ts`) | DA identity, heartbeat, scheduling, growth | SOUL.md + Hermes cron + Hindsight | See DA Subsystem section below. |
| **UserIndex** (`modules/user-index.ts`) | USER/ indexer with fs.watch | Hindsight recall + cognitive-graph | No filesystem watcher. Recall is the index. |

Key architectural difference: Pulse was a monolithic daemon (one process, one port, one launchd plist). Hermes distributes these responsibilities across its native runtime: cron, plugins, gateway, TTS, LCM, and the desktop app itself.

## DA Subsystem → Hermes-Native

The DA subsystem formalized how LifeOS instantiated a Digital Assistant. In Hermes, the DA is already instantiated — it's the agent running this session, with identity in SOUL.md.

| DA Component | LifeOS Function | Hermes Equivalent |
|---|---|---|
| **Identity Registry** (`_registry.yaml`, `DA_IDENTITY.md`) | Multi-DA management with schema | `$HERMES_HOME/SOUL.md` — single DA identity. Multi-DA not supported in Hermes. |
| **Heartbeat** (2-layer: deterministic + Haiku) | Proactive "should I do something?" every 30 min | Hermes cron job — `cronjob` with a prompt that evaluates current state and acts. No separate heartbeat module. |
| **Scheduler** (`scheduled-tasks.jsonl`) | Natural language → cron | Hermes cron + `cronjob` tool. Natural language routing via the DA itself. |
| **Growth Engine** (diary, opinions, drift) | Identity evolution tracking | Hindsight `cat:identity` + `hindsight_reflect` via cron. No separate growth.jsonl. |
| **Interview System** (`DAInterview.ts`) | Guided DA creation wizard | Interview skill (already ported). SOUL.md is the output. |
| **Multi-DA** (primary + worker) | Shared infrastructure, distinct personalities | Not ported. Hermes has one DA per profile. |

### Heartbeat → Hermes Cron

The LifeOS heartbeat was a 2-layer architecture: Layer 1 ($0, deterministic context gathering) → Layer 2 (~$0.001, Haiku evaluation). Most ticks returned `NO_ACTION`.

In Hermes, this maps to a cron job that runs periodically (e.g., every 30 min) with a self-contained prompt:
- Layer 1 is the cron prompt itself — it gathers context via Hindsight recall, Conduit rollup, TELOS check.
- Layer 2 is the agent evaluating whether to act.
- Cost is controlled by the cron job's model override and the fact that most ticks produce no user-facing output.

### Growth → Hindsight + Reflect

| LifeOS Growth Mechanism | Hermes Equivalent |
|---|---|
| **Diary** (daily 11PM) | Cron job that calls `hindsight_retain` with `cat:identity` to record the day's notable events |
| **Opinions** (weekly Sunday 4AM) | Cron job that calls `hindsight_reflect` to synthesize confidence-weighted beliefs |
| **Identity drift** (monthly) | SOUL.md updates with explicit user confirmation. No automatic personality drift. |

## Terminal Tab State → Hermes Desktop

LifeOS used Kitty terminal tab colors and title suffixes for instant visual state feedback. Hermes has its own desktop app with panes (chat, files, terminal, review, sessions).

| LifeOS Tab State | Kitty Display | Hermes Equivalent |
|---|---|---|
| **Inference** (🧠, purple) | Brain icon + `…` suffix | Hermes chat pane shows "thinking" indicator natively |
| **Working** (⚙️, orange) | Gear icon + italic text | Hermes shows tool execution in chat pane |
| **Completed** (✓, green) | Checkmark | Hermes chat pane returns to ready state |
| **Awaiting Input** (❓, teal) | Bold caps + question icon | Hermes `clarify` tool renders selectable choices in chat |
| **Error** (⚠, orange) | Warning + `!` suffix | Hermes surfaces errors in chat pane + terminal output |

### Algorithm Phase Tab System

LifeOS drove tab titles/colors through ISA phases (OBSERVE, THINK, PLAN, BUILD, EXECUTE, VERIFY, LEARN, COMPLETE). In Hermes, the Algorithm skill owns phase tracking internally — no tab painting is needed. The chat pane shows phase progress through the agent's own output.

## Pulse Metadata Surface → Hermes Status

Pulse surfaced session metadata through badges, strips, and panels. Hermes has a different surface model:

| Pulse Metadata | Hermes Equivalent |
|---|---|
| **Lifecycle pill** (scoping/climbing/learning/done) | Agent's own status reporting + LCM session state |
| **Rework badge** (iteration count) | ISA iteration count in workspace ISA.md |
| **ClimbChart** (ISC progress over time) | ISA criteria checklist in workspace + `check.py` |
| **QuickPulseStrip** (live ratings, 24h) | Hindsight recall of recent sessions + satisfaction signals |
| **IntensityBar** (tool-call rate) | Hermes terminal/chat shows tool activity natively |
| **JourneyStrip** (current → ideal state) | Constitution §1 + Thesis skill core loop |
| **GoalBadge** | TELOS goal context via Hindsight recall |
| **DensityBadge** | Not ported. No equivalent density/divergence metric in Hermes. |

### Tooltips

Pulse tooltips taught the dashboard's meaning through hover-context. In Hermes, the DA explains metrics through its own responses — the conversational interface replaces hover tooltips. The Freshness skill's text/JSON output includes explanatory context inline.

## Observatory Dashboard → Hermes Terminal + LCM

The Observatory was a Next.js static export served by Pulse on `localhost:31337`. In Hermes:

| Observatory Page | Hermes Equivalent |
|---|---|
| `/agents` (work dashboard) | Hermes sessions pane + LCM session list |
| `/knowledge` (knowledge browser) | `hindsight_recall` with `cat:knowledge` |
| `/security` (security management) | Hermes tool approval + safety middleware (no separate UI) |
| `/freshness` | `python LifeOS/install/skills/Freshness/Tools/check.py` |
| `/growth` | Hindsight `cat:identity` recall + cron `lifeos-wisdom-synthesis` |
| `/life` (USER/ browser) | `hindsight_recall` with category-specific tags |

## Session State Tracking → Hermes Session + LCM

LifeOS tracked session state in `MEMORY/STATE/work.json` (canonical file, atomic read-modify-write). In Hermes:

| LifeOS Session State | Hermes Equivalent |
|---|---|
| `work.json` session registry | LCM session database + Hermes session state |
| ISA session (starting/native) | Hermes workspace ISA.md files |
| Staleness thresholds (5 min native, 10 min algorithm) | LCM session lifecycle |
| Self-healing (Math.max(updatedAt, lastToolActivity)) | LCM handles this natively |

## Removed Components

| Component | Reason |
|---|---|
| Pulse daemon (port 31337) | No always-on daemon in Hermes. Responsibilities distributed. |
| launchd plist (`com.lifeos.pulse`) | macOS-specific. Hermes uses its own process management. |
| Kitty terminal tab painting | Hermes desktop app has its own UI. No Kitty dependency. |
| Next.js Observatory dashboard | No web dashboard. Hermes desktop app is the surface. |
| Mode/tier token (N, E1-E5) | Retired 2026-07-11 in LifeOS. Not applicable to Hermes. |
| effort-router.jsonl | Retired 2026-07-11. No effort routing in Hermes. |
| UserIndex fs.watch indexer | Hindsight recall replaces the filesystem indexer. |

## Cross-references

- **`HERMES_CONSTITUTION.md`** — §1 (Operating aim), §6 (Layer boundaries)
- **Thesis skill** — Three-layer model (DA → Pulse → LifeOS in Hermes terms)
- **Observability skill** — Event pipeline mapping
- **Notifications skill** — Voice and push notification mapping
- **Freshness skill** — A-F staleness grading
- **Memory skill** — Hindsight as canonical memory layer
- **`LifeOS/install/LIFEOS/DOCUMENTATION/Pulse/PulseSystem.md`** — Source of truth (do not modify)
