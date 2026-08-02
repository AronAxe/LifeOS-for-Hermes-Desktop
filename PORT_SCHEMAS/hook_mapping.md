# LifeOS to Hermes Hook Mapping Schema

This document maps all legacy LifeOS hooks (from `LifeOS/install/hooks/` and `hooks.json`) to their Hermes-native runtime equivalents.

## Hook Mapping Table

| Hook | Trigger | Hermes-native | Port? | Notes |
|---|---|---|---|---|
| `Safety.hook.ts` | `PostToolUse (WebFetch, WebSearch), PermissionRequest (Write, Edit, MultiEdit, Bash, mcp__.*)` | Hermes tool approval + safety middleware | Yes | Enforces command safety and tool permission gating |
| `PreToolGuard.hook.ts` | `PreToolUse (Bash, Write, Edit, MultiEdit)` | Hermes tool approval + path protection | Yes | Includes `CommunicationSkillGuard`, `SystemFileGuard`, `EgressClassGuard` |
| `ISASync.hook.ts` | `PostToolUse (Write, Edit, MultiEdit)` | Hermes post-tool lifecycle → workspace state sync | Yes | Synchronizes ISA state after filesystem modifications |
| `ISARenderOnStop.hook.ts` | `Stop` | Hermes HTML render on completion | Optional | Renders ISA visual dashboard on turn completion |
| `CheckpointPerISC.hook.ts` | `PostToolUse (Write, Edit, MultiEdit)` | Hermes git checkpoint on ISC change | Optional | Automatic git commits on significant file modifications |
| `LoadContext.hook.ts` | `SessionStart` | SOUL.md + `ephemeral_system_prompt` + Hindsight recall | Yes | Injects core persona, context, and recall into initial prompt |
| `MemoryTurnStart.hook.ts` | `UserPromptSubmit` | Hindsight recall via `MemoryManager.prefetch_all()` | Yes | Combines `LoadMemory` and `MemoryDeltaSurface` for memory retrieval |
| `MemoryReviewFire.hook.ts` | `Stop` | Hindsight retain via `MemoryManager.sync_all()` | Yes | Triggers background memory extraction on response complete |
| `MemoryHealthGate.hook.ts` | `SessionEnd` | Hindsight health check (built into plugin) | Yes | Validates memory DB integrity and connection status |
| `LastResponseCache.hook.ts` | `Stop` | Hermes inter-turn state bridge (built-in) | Yes | Caches final output for inter-turn state continuity |
| `StopGates.hook.ts` | `Stop` | Hermes turn-completion middleware | Yes | Combines `VerificationGate` and `WritingGate` for turn compliance |
| `PostToolObserver.hook.ts` | `PostToolUse` | Hermes tool-loop governance | Yes | Includes `LoopDetector` to detect repeating tool calls |
| `AgentInvocation.hook.ts` | `PreToolUse (Agent), PostToolUse (Agent)` | Hermes delegation lifecycle events | Yes | Intercepts subagent spawns and completions |
| `TaskGovernance.hook.ts` | `TaskCreated` | Hermes subagent rate limiting | Yes | Governs concurrent agent limits and subtask throttling |
| `SessionCleanup.hook.ts` | `SessionEnd` | Hermes session teardown | Yes | Cleans temporary scratch files and transient sockets |
| `WorkCompletionLearning.hook.ts` | `SessionEnd` | Hindsight retain + cognitive-graph capture | Yes | Extracts heuristics and learnings into `mind.db` graph |
| `EventLogger.hook.ts` | `PostToolUse, PostToolUseFailure, ConfigChange, StopFailure` | Hermes telemetry/logging layer | Yes | Structured logging of execution events and errors |
| `HookHealer.hook.ts` | `SessionStart` | Not needed (Hermes plugin loader handles this) | No | Auto-heals missing bun hooks in Claude Code |
| `IntegrityCheck.hook.ts` | `SessionEnd` | Hermes session integrity (built-in) | Yes | Validates system state consistency before shutdown |
| `DocIntegrity.hook.ts` | `SessionEnd` | Hermes post-session doc maintenance | Optional | Audits system doc consistency on session exit |
| `AlgorithmNudge.hook.ts` | `PostToolUseFailure` | Hermes skill routing + error recovery | Yes | Constitution guides skill routing and error recovery |
| `ReminderRouter.hook.ts` | `UserPromptSubmit` | Hermes intent interceptor | Optional | Routes pending user reminders into prompt pipeline |
| `SatisfactionCapture.hook.ts` | `UserPromptSubmit` | Hindsight retain on feedback | Yes | Evaluates user satisfaction markers for learning |
| `ContextReduction.hook.sh` | `PreToolUse (Bash)` | Not needed | No | Hermes manages context window natively |
| `TabState.hook.ts` | `PreToolUse (AskUserQuestion), PostToolUse (AskUserQuestion), Stop` | N/A | No | Kitty terminal tab titles (Claude-only) |
| `PromptProcessing.hook.ts` | `UserPromptSubmit` | N/A | No | Kitty terminal titles during prompt processing (Claude-only) |
| `VoiceCompletion.hook.ts` | `Stop` | Hermes TTS plugin | No | Replaced with native Hermes TTS plugin |
| `KittyEnvPersist.hook.ts` | `SessionStart` | N/A | No | Kitty environment persistence (Claude-only) |
| `UpdateCounts.hook.ts` | `SessionEnd` | N/A | No | Claude Code status bar banner metadata (Claude-only) |
| `FormatGate.hook.ts` | (Unregistered / legacy) | N/A | No | Legacy unregistered formatting filter |
| `DriftReminder.hook.ts` | (Unregistered / legacy) | Hermes constitution | No | Replaced by constitutional prompt adherence |
| `com.lifeos.amberroute` | `launchd` every 30min | Hermes cron `amber-route` every 30min | Yes | Grades unrouted Amber captures against TELOS via Hindsight recall+retain, routes to destinations |
| `com.lifeos.conduit` | `launchd` every 120s | Hermes cron `conduit-capture` every 2min | Yes | Runs `capture.py` to poll current-state signals (appFocus/git/hermesSession) into events.jsonl |
| `com.lifeos.conduit.insight` | `launchd` every 1h | Hermes cron `conduit-rollup` daily | Yes | Runs `rollup.py` for deterministic daily record, retains into Hindsight |

## Tools, CLI & Containment additions

| Source component / doctrine | Hermes-native | Port? | Notes |
|---|---|---|---|
| `Tools__CliFirstArchitecture` | `CliFirstArchitecture` skill | Yes — doctrine | Deterministic executable operations precede prompts. Hermes consumes local capabilities through native tools, scripts, or CLI; MCP serves a capability only when external clients need it. |
| `Tools__Cli` / legacy Arbol `pai` action-pipeline model | `CLI` skill + `terminal`, `execute_code`, durable scripts, `cronjob` | Adapted | Preserve explicit input/output, stream separation, exit status, JSON-compatible composition, and direct verification. The custom runner, Bun, and model-level wrappers are retired dependencies. |
| `Tools__Tools` utility inventory | `Tools` skill + native Hermes tools | Yes — doctrine | `Inference` → configured Hermes provider/tools; memory retrieval → Hindsight/LCM; graph → cognitive graph; monitor → process/cron; doctor → live probes. No custom utility clone. |
| `Tools__Containment` + `SystemFileGuard.hook.ts` | `Containment` skill + constitution §8 + deliberate staging/release review | Adapted | The portable/private boundary is retained, but Hermes has no direct equivalent of the source system's static containment-zone registry or write-time Claude hook. It is an explicit review gate, not a claimed automatic enforcement layer. |

## Config & Delegation additions

| Component | Trigger | Hermes-native | Port? | Notes |
|---|---|---|---|---|
| `SystemFileGuard.hook.ts` | `PreToolUse (Write, Edit, MultiEdit)` | Hermes tool approval + path protection | Yes | Already covered as a sub-component of `PreToolGuard.hook.ts` above; the LifeOS system/user write-time guard maps to Hermes tool gating + the constitution's "confirm scope/destination/reversibility" invariant. See the **Config** skill (five-layer layering) for the boundary it enforces. |
| `MergeSettings.ts` | `SessionStart` (LifeOS settings merge driver) | — | No | Not ported. Hermes reads `config.yaml` natively; there is no `settings.system.json` + `settings.user.json` → generated `settings.json` merge. The **Config** skill documents the Hermes-native layering that replaces it. |
| Delegation model injection (`AgentInvocation.hook.ts`, retired 2026-07-11) | `PreToolUse (Agent)` | Hermes delegation config (`delegation.*` in `config.yaml`) | No | The LifeOS hook that injected per-agent model choice is retired. On Hermes, subagent model/provider/effort/concurrency come from `delegation.*` config and per-dispatch `delegate_task(..., model=...)`. See the **Delegation** skill + `AgentReference.md`. (Distinct from the `AgentInvocation.hook.ts` lifecycle-events row above, which maps spawn/completion events.) |

## ISA & Freshness additions

| Component | Trigger | Hermes-native | Port? | Notes |
|---|---|---|---|---|
| `ISASync.hook.ts` render trigger | `PostToolUse (phase:complete → ISARender)` | Manual `/render-isa` or `python Tools/render.py` | No | The Claude completion-gate auto-render is not ported. Hermes uses explicit invocation. ISA.md is authoritative; the mirror is derived. |
| `ISARenderOnStop.hook.ts` | `Stop` (batch HTML render on turn-end) | — | No | Not ported. No Hermes equivalent for automatic turn-end HTML re-render. The mirror fires on manual invocation only. |
| `TelosFreshness.ts` | `SessionStart` / CLI / Pulse routes | `Freshness/Tools/check.py` + Freshness skill | Yes | A-F grading ported to Python stdlib. Reads TELOS Dropbox file mtimes + SOUL.md. No Pulse routes — JSON/text output. |
| `FreshnessCache.ts` | Statusline cache (Pulse `/api/freshness/summary`) | — | No | Not ported. Hermes has no persistent terminal statusline. Freshness is queried on demand via the skill or `check.py`. |

## Memory, Schema & Thesis additions

| Component | Trigger | Hermes-native | Port? | Notes |
|---|---|---|---|---|
| `MutationTier.ts` | Memory write | `Memory` skill + Hindsight `retain` with tier-appropriate tags | Yes | Four-tier mutation system mapped to Hindsight layer boundaries (A=auto-retain set-replace, B=append unique, C=propose+confirm, D=never) |
| `MemoryGraph.ts` | Memory graph traversal | `hindsight_recall` + cognitive-graph | Yes | Replaced by Hindsight recall and cognitive-graph typed edges |
| `MemoryRetriever.ts` | Recall at turn start | `hindsight_recall` via `MemoryManager.prefetch_all()` | Yes | Already mapped in MemoryTurnStart row above |
| `MemoryWriter.ts` | Retain at turn end | `hindsight_retain` via `MemoryManager.sync_all()` | Yes | Already mapped in MemoryReviewFire row above |
| `MemoryReviewer.ts` | 8 turns / 30 min / 2 idle | Hermes turn lifecycle (turn-end retain + async `hindsight_reflect`) | Yes | Cadence built into turn lifecycle, not a subprocess |
| `MemoryInsights.ts` | Periodic synthesis | `hindsight_reflect` via cron `lifeos-wisdom-synthesis` (every 6h) | Yes | Async reflection replaces periodic insight subprocess |
| `MemoryStatus.ts` / `MemoryHealthCheck.ts` | SessionEnd health gate | Hindsight plugin health check (built-in) | Yes | Already mapped in MemoryHealthGate row above |
| `MemoryRestore.ts` | Session restore | LCM session recovery + Hindsight recall | Yes | LCM handles transcript continuity; Hindsight handles durable memory |
| `LifeOsSchema` (USER/ directory) | File watch / indexer | `Schema` skill + Hindsight tags + cognitive-graph | Yes | File-tree schema → Hindsight document mapping. No fs.watch indexer. |
| `LifeOsThesis` | Conceptual reference | `Thesis` skill | Yes | Operational adaptation of the thesis for Hermes-native DA. Not a runtime component — reference knowledge. |

## Pulse, Notifications & Observability additions

| Component | Trigger | Hermes-native | Port? | Notes |
|---|---|---|---|---|
| `pulse.ts` / `pulse-unified.ts` | Always-on daemon (port 31337) | Hermes cron + plugins + gateway + TTS + LCM | Yes | Monolithic daemon → distributed runtime. No single process or port. |
| `VoiceServer/voice.ts` | `/notify` endpoint → ElevenLabs | Hermes TTS plugin (`text_to_speech`) | Yes | curl→/notify→VoiceServer chain replaced by single tool call |
| `Observability/observability.ts` | HTTP API + Next.js dashboard | Hermes LCM (`lcm_status`, `lcm_inspect`, `session_search`) + desktop app | Yes | JSONL event files → LCM session DB. No polling. |
| `modules/telegram.ts` | grammY polling bot | Hermes gateway integration | Yes | Native Hermes gateway handles Telegram |
| `modules/imessage.ts` | SQLite polling bot | Not ported (Windows) | No | iMessage not available on Windows |
| `checks/github-work.ts` | GitHub Issues polling | Hermes cron + `gh` CLI | Yes | Cron job that runs `gh issue list` periodically |
| `Assistant/module.ts` (DA subsystem) | DA identity, heartbeat, growth | SOUL.md + Hermes cron + Hindsight | Yes | Identity in SOUL.md, heartbeat as cron job, growth via `hindsight_reflect` |
| `modules/user-index.ts` | USER/ indexer with fs.watch | Hindsight recall + cognitive-graph | Yes | No filesystem watcher. Recall is the index. |
| `modules/hooks.ts` | Skill-guard and agent-guard | Hermes tool approval + safety middleware | Yes | Built into Hermes runtime |
| `notifications.ts` (smart routing) | Event type → channel routing | DA judgment + `send_message(target="phone")` + `text_to_speech` | Yes | Routing table → DA judgment. Most events stay in chat. |
| `lib/homographs.ts` / `PRONUNCIATIONS.json` | Pronunciation normalization | Not ported | No | Hermes TTS plugin handles pronunciation natively |
| `VoiceCompletion.hook.ts` | Stop → extract `🗣️` → `/notify` | Hermes TTS plugin (built-in) | Yes | No hook needed. DA decides when to speak. |
| `notification-governor.ts` | Notification rate limiting | DA judgment | Yes | No separate governor. DA avoids notification fatigue by design. |
| `EventLogger.hook.ts` | PostToolUse → JSONL append | Hermes built-in tool logging + LCM | Yes | No JSONL files. LCM and session DB track events natively. |
| `AgentInvocation.hook.ts` | Pre/PostToolUse(Agent) → subagent-events.jsonl | Hermes `delegate_task` lifecycle tracking | Yes | Native delegation tracking. No JSONL. |
| `MEMORY/STATE/work.json` | Session state registry | LCM session database | Yes | Atomic read-modify-write → LCM native session management |
| `SessionAnalysis.hook.ts` | UserPromptSubmit → upsertSession | Hermes session start (LCM) | Yes | Session classification built into LCM |
| `tab-setter.ts` / `TabState.hook.ts` | Kitty tab painting | Hermes desktop app panes | No | Kitty terminal not used. Desktop app has its own UI. |
| `PromptProcessing.hook.ts` | UserPromptSubmit → working state | Hermes chat pane (native) | No | No tab painting needed. Chat pane shows state natively. |
| `KittyEnvPersist.hook.ts` | SessionStart → Kitty env | Not needed | No | No Kitty terminal in Hermes |

## Router, Security & BackgroundServices additions

| Component | Trigger | Hermes-native | Port? | Notes |
|---|---|---|---|---|
| `TheRouter.hook.ts` (retired 2026-07-11) | `UserPromptSubmit` → classify mode/tier | DA judgment + skill loading | No | Retired. No classifier hook. DA calibrates effort by choosing what to load, what to delegate, and how many passes to run. See **Router** skill. |
| `LIFEOS/TOOLS/models.ts` `EFFORT_MODEL` | Model selection by effort level | `config.yaml` `model.default` + provider config | No | No four-level abstraction. Single default model per profile. See **Router** skill. |
| `LIFEOS/TOOLS/Inference.ts` | Utility-path inference (summaries, classification, vision) | Direct tool calls (`web_search`, `web_extract`, `vision_analyze`) | No | No separate utility-inference layer. Tools run directly. |
| `LIFEOS/ALGORITHM/v{VERSION}.md` tier→level table | Effort routing policy | DA judgment + Algorithm skill | No | No fixed tier→level table. DA calibrates effort implicitly. |
| `additionalContext` classifier contract | Classifier output → executor | Hindsight recall + LCM + skills + constitution | No | No classifier output block. Context comes from Hermes runtime mechanisms. |
| `EgressClassGuard.hook.ts` | `PreToolUse` → egress class check | Provider selection + DA judgment + constitution §8 | No | No runtime egress guard. DA classifies data sensitivity and selects trusted providers. See **Security** skill. |
| `Safety.hook.ts` (PermissionRequest) | Outgoing tool call shape classification | Hermes tool approval + DA judgment | Yes | Shape catalog encoded in DA security knowledge, not regex. See **Security** skill. |
| `Safety.hook.ts` (PostToolUse/annotation) | Web content → `[EXTERNAL CONTENT]` header | Constitution §8 (DA judgment) | No | DA treats external content as data per constitution. No header injection needed. |
| `safety-classifier.ts` shape catalog | Regex patterns for dangerous/credential/injection | DA knowledge + Hermes tool gating | No | Patterns encoded in DA's security knowledge, not a regex library. |
| `permission-cache.json` | SHA-keyed allow-only cache | Not needed | No | Hermes tool approval is stateless per call. |
| `permission-decisions.jsonl` / `egress-decisions.jsonl` | Telemetry logging | LCM + session DB | Yes | No JSONL telemetry. LCM tracks natively. |
| `DENY_LIST.txt` + `DenyListCheck.ts` + `ShadowRelease.ts` | Release pipeline sensitive-pattern gating | DA judgment + git hygiene | No | No automated release pipeline. DA checks for sensitive patterns before publishing. |
| `ContainmentGuard.hook.ts` (retired) | PreToolUse → containment zone check | `Containment` skill + constitution §8 + staged release review | Adapted | Retains the portable/private policy but not the static zone registry or automatic pre-write hook. Before public commits/releases, inspect the actual diff and scan the intended public files for personal data, credentials, private identifiers, and user-specific paths. |
| `LIFEOS/TOOLS/Services.ts` | Service registry + `launchd` install/uninstall | `cronjob action='list'/'create'/'remove'` + `process(action='list')` | Yes | One registry + launchd → Hermes cron + background processes. See **BackgroundServices** skill. |
| `~/Library/LaunchAgents/*.plist` | `launchd` job definitions | `cronjob` schedule config | Yes | Plists → cron schedule expressions. Cross-platform (no macOS dependency). |
| `launchctl list` | Live service state | `cronjob action='list'` + `process(action='list')` | Yes | Live state from runtime, not a file. |
| `com.lifeos.synthesis` | `launchd` daily synthesis pass | `hindsight_reflect` via cron `lifeos-wisdom-synthesis` (every 6h) | Yes | Async reflection replaces periodic synthesis subprocess. |
| `com.lifeos.worksweep` | `launchd` every 1h work capture | Hermes cron (optional, not currently configured) | Optional | Can review session state and TELOS alignment. |
| `com.lifeos.derivedsync` | `launchd` file-change sync (31 files) | Not needed | No | No fs.watch indexer. Hindsight recall is the index. See **Schema** skill. |
| `com.lifeos.healthsync` | `launchd` every 1h health sync | Hermes cron (optional) | Optional | Can sync health data if needed. Not currently ported. |
| `com.lifeos.commitmentsweep` | `launchd` daily commitment sweep | Hermes cron (optional) | Optional | Can check Hindsight for commitments/reminders. |
| `com.lifeos.blogdiscovery` | `launchd` daily blog signal | Hermes cron (optional) | Optional | Can discover blog-worthy signal from recent work. |
| `com.lifeos.usage-aggregator` | `launchd` daily usage telemetry | Not needed | No | Hermes tracks usage natively via LCM. |
| `com.lifeos.backups` | `launchd` daily 03:00 repo backup | Git push (manual or cron) | Optional | Repo backed up via git. Cron can automate. |
| Hourly security scanner (Arbol) | Scheduled external security scan | DA judgment + manual review | No | No automated scanner. DA performs security review before publishing. |

## SkillSystem, Synapse & Testing additions

| Component | Trigger | Hermes-native | Port? | Notes |
|---|---|---|---|---|
| `skills/CreateSkill/SKILL.md` + `Skills__SkillSystem` doctrine | Skill creation, canonicalization, discovery | `skills_list`, `skill_view`, `skill_manage`, `read_file` | Yes | LifeOS naming, routing, customization, and ideal-state doctrine mapped to Hermes skill operations. No Claude settings or file-memory runtime. |
| `skills/_*/**` private boundary | Release containment | Hermes skill provenance + explicit private/public review | Yes | TitleCase skills remain generic; identity-bound or environment-specific content is private and must not enter public artifacts. |
| `SKILLCUSTOMIZATIONS` / `PREFERENCES.md` | Per-user overlays | Hindsight durable preferences + generic skill body | Adapted | Hindsight stores durable preferences/knowledge; skills remain generic. |
| Synapse capture contract | Any input crosses attention | Hindsight `retain` with stable `user:{user_id}:synapse:{capture_id}` and provenance tags | Yes | Preserve before grade; `source`, `external_id`, `url`/`content`, `captured_at`, `content_kind`, `privacy_class` are required. |
| Amber ledger / `amber` D1 journal | Write-ahead preservation | Hindsight durable capture records | Adapted | Amber is the legacy name; Synapse is canonical. No custom D1 or JSONL runtime. |
| Synapse grade | TELOS alignment and quality evaluation | Hindsight `recall`/`reflect` + DA judgment | Yes | Weak captures remain preserved even when no destination is earned. |
| Synapse route | Destination selection | Hindsight categories, cognitive graph, workspace/ISA, approved external outputs | Yes | Active task state remains workspace/ISA, not Hindsight. |
| LifeOS `bun test` / shared Bun harness | ISA `## Test Strategy` → executable probe | Hermes native repository tests, `terminal`, `read_file`, `git diff`, provider read-back | Adapted | Bun harness is not a Hermes dependency. Match the probe to the artifact and never fabricate green evidence. |
| `Anti:` ISC | Regression prevention | Testing skill + ISA test strategy | Yes | Anti-criteria are first-class and must be directly falsified by a real probe. |
| State-machine edge tests | Transition verification | Hermes Testing skill | Yes | The test must produce the transition; hand-written terminal state is insufficient. |