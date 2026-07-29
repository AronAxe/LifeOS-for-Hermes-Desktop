---
name: background-services
category: LifeOS
description: >
  Use when reasoning about recurring background services, scheduled jobs, daemon
  lifecycles, or the mapping from LifeOS launchd services to Hermes cron and
  plugins. One registry concept → distributed Hermes-native scheduling.
---

# Background Services Skill — LifeOS Services → Hermes-Native Mapping

## What This Covers

LifeOS ran every recurring Mac service through one registry: `LIFEOS/TOOLS/Services.ts`. This skill maps that registry concept to Hermes-native equivalents (cron, plugins, gateway, and background processes).

## The LifeOS Model

### The one-shot tool

```bash
bun ~/.claude/LIFEOS/TOOLS/Services.ts status              # what's running vs installed vs available
bun ~/.claude/LIFEOS/TOOLS/Services.ts install --all --yes  # stand up every service (macOS)
bun ~/.claude/LIFEOS/TOOLS/Services.ts install --only worksweep,amberroute --yes
bun ~/.claude/LIFEOS/TOOLS/Services.ts uninstall --only amberroute
bun ~/.claude/LIFEOS/TOOLS/Services.ts doc                 # regenerate the table
```

`Services.ts` is a single registry (`SERVICES`) of human-meaningful metadata — title, purpose, category, opt-in, install command — merged at runtime with the actual plists on disk. Cadence and run-state are parsed live from `~/Library/LaunchAgents/*.plist` and `launchctl list`, so `status` reports reality, not a stale hand-maintained list.

### The service registry

| Service | Category | Cadence | Opt-in | Purpose |
|---|---|---|---|---|
| **Pulse** (dashboard server) | pulse | at load | core | HTTP server on :31337 — the visible LifeOS surface |
| **Pulse menu-bar app** | pulse | at load | core | macOS menu-bar status + open dashboard |
| **Pulse deriver** | pulse | daily | core | Regenerates derived Data-Plane pages |
| **Conduit** (sensory capture) | capture | every 2m | core | Local current-state capture → memory + TELOS |
| **Conduit insight builder** | capture | every 1h | core | Builds insights from Conduit's captured signal |
| **Synthesis** | maintenance | daily | yes | Periodic synthesis pass over recent state/memory |
| **Work sweep** | sweep | every 1h | yes | Untracked sessions, stale items, project checks, TELOS-goal derivation |
| **Conveyor inbox watcher** | capture | KeepAlive | yes | Watches `~/Recordings/Inbox`; hash-idempotent registration |
| **Derived-file sync** | sync | file-change | yes | Watches 31 USER source files; regenerates derived files |
| **Health sync** | sync | every 1h | yes | Syncs health data into CURRENT_STATE |
| **Codex update** | maintenance | daily | yes | Keeps Codex mirror / update state current |
| **Commitment sweep** | sweep | daily | yes | Sweeps commitments/reminders |
| **Blog discovery** | sweep | daily | yes | Discovers blog-worthy signal |
| **Usage aggregator** | maintenance | daily | yes | Aggregates usage/cost telemetry for Pulse |
| **Bookmark pipeline watchdog** | capture | every 4h | yes | Watches X bookmark → summarize/idea pipeline |
| **Backups** | maintenance | daily 03:00 | yes | Daily repo backup (Git LFS) |
| **Amber router** | capture | every 30m | yes | TELOS-grade unrouted Amber captures → KNOWLEDGE notes / UL issues |

### Design principle: one registry, live state

The registry supplies human-meaningful metadata; plists and `launchctl` supply ground truth. Because state comes from the live system every time, the table cannot rot the way a hand-maintained service list would. Adding a service = one row in `SERVICES` pointing at its existing installer; the tool orchestrates, it does not re-implement each installer.

## Hermes-Native Mapping

### The distributed equivalent

LifeOS used a single `Services.ts` registry + `launchd` for all recurring services. Hermes distributes this across multiple native mechanisms:

| LifeOS | Hermes-native | Mechanism |
|---|---|---|
| `Services.ts` registry | `cronjob action='list'` | Hermes cron lists all scheduled jobs. No separate registry file. |
| `launchd` plists | `cronjob action='create'` | Each service is a cron job with schedule, prompt, optional skills, and delivery target. |
| `launchctl` live state | `cronjob action='list'` + `process(action='list')` | Cron list shows job state. Process list shows running background terminals. |
| `Services.ts install/uninstall` | `cronjob action='create'/'remove'` | Creating/removing cron jobs replaces install/uninstall. |
| `Services.ts status` | `cronjob action='list'` + `process(action='list')` | Combined view of scheduled jobs and running processes. |
| `Services.ts doc` | This skill file | The mapping table below replaces the generated doc. |
| macOS-only `launchd` | Cross-platform | Hermes cron runs on any platform. No `launchd` dependency. |

### Service-by-service mapping

| LifeOS service | Hermes-native | How |
|---|---|---|
| **Pulse** (dashboard server) | Hermes desktop app | The desktop app IS the dashboard. No HTTP server to run. See **Pulse** skill. |
| **Pulse menu-bar app** | Hermes desktop app (native) | Desktop app has its own status indicators. No menu-bar app needed. |
| **Pulse deriver** | Hermes cron (optional) | If derived views are needed, a cron job can regenerate them. Currently not ported — desktop app shows live state. |
| **Conduit** (sensory capture) | Hermes cron `conduit-capture` | Cron job every 2min running `capture.py`. See **Conduit** skill. Already mapped in `hook_mapping.md`. |
| **Conduit insight builder** | Hermes cron `conduit-rollup` | Daily rollup cron job. Already mapped in `hook_mapping.md`. |
| **Synthesis** | `hindsight_reflect` via cron | Async reflection replaces periodic synthesis subprocess. See **Memory** skill. |
| **Work sweep** | Hermes cron (optional) | Can be implemented as a cron job that reviews session state and TELOS alignment. Not currently configured. |
| **Conveyor inbox watcher** | Not ported | Recording ingestion is not part of the Hermes port. Manual file handling. |
| **Derived-file sync** | Not needed | No fs.watch indexer. Hindsight recall is the index. See **Schema** skill. |
| **Health sync** | Hermes cron (optional) | Can be configured if health data sync is needed. Not currently ported. |
| **Codex update** | Not ported | Codex-specific. Not relevant to Hermes. |
| **Commitment sweep** | Hermes cron (optional) | Can be implemented as a cron job that checks Hindsight for commitments/reminders. |
| **Blog discovery** | Hermes cron (optional) | Can be implemented as a cron job that discovers blog-worthy signal from recent work. |
| **Usage aggregator** | Not needed | Hermes tracks usage natively via LCM and session DB. No separate aggregator. |
| **Bookmark pipeline watchdog** | Not ported | X bookmark pipeline is not part of the Hermes port. |
| **Backups** | Git + manual/cron | The repo is backed up via git push. A cron job could automate this. |
| **Amber router** | Hermes cron `amber-route` | Already mapped in `hook_mapping.md`. Grades unrouted Amber captures against TELOS. See **Amber** skill. |

### Already-ported services (in hook_mapping.md)

Three LifeOS launchd services have already been mapped to Hermes cron in `PORT_SCHEMAS/hook_mapping.md`:

| LifeOS service | Hermes cron | Cadence | Status |
|---|---|---|---|
| `com.lifeos.amberroute` | `amber-route` | every 30min | Mapped |
| `com.lifeos.conduit` | `conduit-capture` | every 2min | Mapped |
| `com.lifeos.conduit.insight` | `conduit-rollup` | daily | Mapped |

### The registry principle in Hermes

LifeOS's key insight — **one registry, live state** — maps to Hermes as:

1. **`cronjob action='list'` is the registry.** It shows all scheduled jobs, their schedules, and whether they are enabled. No separate file to maintain.
2. **`process(action='list')` is the live state.** It shows what is actually running right now.
3. **Adding a service = `cronjob action='create'`.** One call creates the job with schedule, prompt, optional skills, and delivery target. No installer script.
4. **Removing a service = `cronjob action='remove'`.** One call removes it. No uninstaller.
5. **State comes from the runtime, not a file.** Cron list and process list query the live system, preventing drift.

### Categories in Hermes

| LifeOS category | Hermes equivalent | Examples |
|---|---|---|
| **pulse** | Desktop app (native) | Dashboard, status |
| **capture** | Cron jobs + scripts | Conduit, Amber router |
| **sweep** | Cron jobs (DA-designed) | Work sweep, commitment sweep |
| **sync** | Not needed or cron (optional) | Derived-file sync → Hindsight recall; health sync → optional cron |
| **maintenance** | Cron jobs + Hindsight | Synthesis → `hindsight_reflect`, backups → git push |

## What is NOT ported

| LifeOS service | Reason |
|---|---|
| Pulse dashboard server | Replaced by Hermes desktop app (native). No HTTP server. |
| Pulse menu-bar app | Desktop app has native status indicators. |
| Pulse deriver | Not needed. Desktop app shows live state. |
| Conveyor inbox watcher | Recording ingestion not part of Hermes port. |
| Derived-file sync | No fs.watch indexer. Hindsight recall is the index. |
| Codex update | Codex-specific. Not relevant. |
| Usage aggregator | Hermes tracks usage natively via LCM. |
| Bookmark pipeline watchdog | X bookmark pipeline not part of Hermes port. |

## Cross-references

- **Pulse** skill — dashboard/DA subsystem mapping
- **Conduit** skill — current-state sensing
- **Amber** skill — idea capture and routing
- **Memory** skill — synthesis and reflection
- **Observability** skill — event pipeline and LCM
- **Constitution** §6 (Layer boundaries) — cron/plugins/gateway as background services
- `PORT_SCHEMAS/hook_mapping.md` — launchd → cron mappings (already done for 3 services)

## Source

- `Services__BackgroundServices` — the LifeOS background services registry and tool
