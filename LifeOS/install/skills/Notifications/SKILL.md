---
name: notifications
trigger: Use when managing voice notifications, push notifications, smart routing, or mapping LifeOS notification channels to Hermes-native equivalents.
---

# Notifications — LifeOS Notification System → Hermes-Native Mapping

## Purpose

The LifeOS Notification System closed the feedback loop: when the system advanced the hill-climb, the principal heard it without having to look. This skill maps every notification channel, routing rule, and voice mechanism to its Hermes-native equivalent.

## Notification Channels → Hermes

| LifeOS Channel | Service | Hermes Equivalent | Status |
|---|---|---|---|
| **Voice** | ElevenLabs via Pulse `/notify` endpoint (port 31337) | Hermes TTS plugin (`text_to_speech` tool) | Active |
| **ntfy** | ntfy.sh mobile push | Hermes `send_message(target="phone")` — phone notification | Active |
| **Discord** | Webhook | Hermes gateway integrations (if configured) | Optional |
| **Desktop** | macOS native alerts | Hermes desktop app notifications | Active |
| **iMessage** | SQLite polling bot | Not available on Windows. Hermes gateway handles messaging. | Not ported |
| **SMS** | Not recommended (A2P 10DLC) | Not ported. `send_message(target="phone")` replaces this. | Not ported |

## Voice Notifications → Hermes TTS

### The LifeOS Pattern

LifeOS used `curl -s -X POST http://localhost:31337/notify` with a JSON payload to trigger voice. The VoiceServer (`voice.ts`) called ElevenLabs with pronunciation normalization (`applyPronunciations()` + `disambiguateHomographs()`).

### The Hermes Pattern

Hermes replaces the curl→/notify→VoiceServer chain with the `text_to_speech` tool:

| LifeOS | Hermes |
|---|---|
| `curl -s -X POST http://localhost:31337/notify -d '{"message": "..."}'` | `text_to_speech(text="...")` |
| ElevenLabs voice_id from `settings.json` | Hermes TTS plugin (configured in config.yaml) |
| Pronunciation normalization (`homographs.ts`, `PRONUNCIATIONS.json`) | Not ported. TTS plugin handles pronunciation natively. |
| Voice completion hook (`VoiceCompletion.hook.ts` → `VoiceNotification.ts`) | Hermes TTS plugin (built-in, no hook needed) |
| Phase-specific voice (algorithm vs main) | Not ported. Single voice configuration. |

### When to Use Voice

| LifeOS Rule | Hermes Adaptation |
|---|---|
| Skip curl for conversational responses | Skip TTS for simple Q&A, greetings, acknowledgments |
| Announce task start with gerund ("Fixing…", "Creating…") | Use `text_to_speech` for significant task starts if appropriate |
| `Workflows/` directory triggers "Executing…" format | Hermes skills don't have Workflows/ directories. Announce skill execution naturally. |
| Effort level determines which phase curls fire | No effort-based voice routing in Hermes. Use judgment. |
| `VoiceCompletion.hook.ts` extracts `🗣️` line | Not ported. The DA decides when to speak. |

### Voice IDs → Hermes TTS Config

| LifeOS | Hermes |
|---|---|
| `{DA_IDENTITY.VOICEID}` (default DA voice) | Hermes TTS plugin configured voice |
| `21m00Tcm4TlvDq8ikWAM` (Priya, artist) | Not ported. Single voice configuration. |
| `~/.claude/settings.json → daidentity.voices.main.voiceId` | `config.yaml → tts` configuration |

## Smart Routing → Hermes Notification Strategy

LifeOS had a routing table that decided which channels received which event types:

| LifeOS Event | Default Channels | Hermes Equivalent |
|---|---|---|
| `taskComplete` | Voice only | No automatic notification. DA reports in chat. |
| `longTask` (> 5 min) | Voice + ntfy | `send_message(target="phone")` for long-running task completion |
| `backgroundAgent` | ntfy | `send_message(target="phone")` when background delegation completes |
| `error` | Voice + ntfy | `send_message(target="phone")` for significant errors |
| `security` | Voice + ntfy + Discord | `send_message(target="phone")` + Hermes security logging |

### Hermes-Native Routing Principles

1. **Chat is the default.** Most notifications stay in the chat pane. The principal sees them when they look.
2. **Phone for important async.** Use `send_message(target="phone")` for: long task completion, background agent completion, errors, security events.
3. **TTS for emphasis.** Use `text_to_speech` for significant milestones when the principal is at the desktop.
4. **No notification fatigue.** Most events resolve to nothing. Notify when real work begins or ends, not on every turn.

## Event Log Channel → Hermes Logging

LifeOS emitted events via `fs.appendFileSync` to `MEMORY/OBSERVABILITY/*.jsonl` — synchronous, fire-and-forget. In Hermes:

| LifeOS Event Log | Hermes Equivalent |
|---|---|
| `tool-activity.jsonl` | Hermes tool execution logs (built-in) |
| `tool-failures.jsonl` | Hermes error logs (built-in) |
| `voice-events.jsonl` | Hermes TTS plugin logs |
| `subagent-events.jsonl` | Hermes delegation logs (`delegate_task` output) |
| `config-changes.jsonl` | Hermes config audit (built-in) |
| `security` events | Hermes security logs |
| `isa-rework.jsonl` | ISA iteration tracking in workspace ISA.md |
| `frame-drift.jsonl` | Not ported. No equivalent drift metric. |
| `reviewer-runs.jsonl` / `reviewer-fires.jsonl` | Hindsight `hindsight_reflect` logs |
| `memory-writes.jsonl` / `tier-b-writes.jsonl` | Hindsight `hindsight_retain` logs |
| `pending-proposals.jsonl` / `identity-proposals.jsonl` | Cognitive-graph + Hindsight (proposals require explicit user confirmation) |
| `proposal-replies.jsonl` | Not ported. Confirmations happen in chat. |

### Key Principle

Hermes does not use JSONL append files for observability. The runtime handles logging natively through LCM, session databases, and built-in tool execution tracking. Do not recreate JSONL event streams.

## ntfy.sh → Hermes Phone Notifications

| LifeOS ntfy | Hermes |
|---|---|
| `ntfy.sh/pai-[random-topic]` | `send_message(target="phone")` |
| Topic name as password | Hermes handles authentication natively |
| `notifications.ntfy.enabled` in settings.json | Phone notifications built into Hermes |
| `notifications.thresholds.longTaskMinutes: 5` | DA judgment — no fixed threshold |

## Discord → Hermes Gateway

| LifeOS Discord | Hermes |
|---|---|
| Webhook in `settings.json` | Hermes gateway integration (if configured) |
| `notifications.discord.enabled` | Gateway configuration |
| Security alerts → Discord | DA uses `send_message` or gateway if configured |

## Pronunciation Normalization

LifeOS had a two-stage normalization before text reached ElevenLabs:

1. `disambiguateHomographs()` — context-aware respelling (e.g., "live" → "lyve" for broadcast/adjective sense only)
2. `applyPronunciations()` — literal term map from `PRONUNCIATIONS.json`

**Not ported.** The Hermes TTS plugin handles pronunciation natively. If pronunciation issues arise, they are handled through the TTS plugin's configuration, not a custom normalization layer.

## Dual-Source Phase Tracking

LifeOS had a dual-source pattern: `/notify` (first-fires, always-fires) and ISASync hook (rich-but-sometimes-skipped) both fed `phaseHistory`. In Hermes, phase tracking is singular — the Algorithm skill owns it internally, and LCM records the transcript. No dual-source reconciliation is needed.

## Cross-references

- **`HERMES_CONSTITUTION.md`** — §6 (Layer boundaries), §8 (Security and external content)
- **Pulse skill** — Daemon architecture and module mapping
- **Observability skill** — Event pipeline and logging
- **Algorithm skill** — Phase tracking (owns the phase lifecycle)
- **`LifeOS/install/LIFEOS/DOCUMENTATION/Notifications/NotificationSystem.md`** — Source of truth (do not modify)
