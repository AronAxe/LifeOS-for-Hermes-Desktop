---
name: router
category: LifeOS
description: >
  Use when reasoning about model selection, effort calibration, or dispatch policy
  in the Hermes-native runtime. Maps the retired LifeOS Router subsystem to
  Hermes config, delegation settings, and DA judgment.
---

# Router Skill — LifeOS Router → Hermes-Native Mapping

## Status: Retired Subsystem (Historical Reference)

The LifeOS Router was **retired on 2026-07-11**. The `TheRouter.hook.ts` classifier, the mode/tier classifier, and the E1–E5 effort tiers no longer exist as runtime components. This skill preserves the conceptual mapping for Hermes-native equivalents. Nothing here is a runtime component — it is reference knowledge for understanding how model selection and effort calibration work in the Hermes port.

## What the Router Was

The Router was the subsystem that decided **how** every prompt got handled — mode, effort tier, stated goal, model rung, and which agent or vendor runs the work. It ran first (classification happened at prompt submit) and kept deciding: each time the work dispatched an agent, the Router picked that agent's model and effort.

The one-line test for what belonged to the Router: **does it help decide *how* to handle a prompt?** If yes, it was the Router. The Router's job was deciding posture — mode, tier, model, effort, agent. Executing that posture — the seven phases, reading files, verification — was the Algorithm (or a NATIVE turn).

### Why it was a subsystem

Three reasons it earned first-class status:

1. **Dynamic range preservation.** Genuinely fast on trivial work, genuinely deep on hard work, with sharp variation between. A greeting gets a reflexive answer; a doctrine redesign gets seven phases, cross-vendor audits, and the top model rung.
2. **Harness limit workaround.** The main conversation loop's model and reasoning effort were fixed at turn start and could not be changed mid-turn. The Router did its heavy lifting by classifying up front and by programming the model and effort of dispatched agents (which were settable per call).
3. **Two edit points for the whole model lineup.** A model entering/leaving the subscription or moving rungs was a one-line change.

### The four stages

Every prompt flowed through four stages, in order:

1. **Classify** (`TheRouter.hook.ts`) — A `UserPromptSubmit` hook turned the raw prompt into a mode and (if ALGORITHM) a tier. Three-stage cascade, cheapest first. Also ran the goal-signal detector and set interview eligibility.
2. **Route the effort** (Algorithm doctrine) — The tier picked an effort level (`max`/`high`/`medium`/`low`) and a dispatch profile. This was policy, stated as intent — the output was a level, never a model name.
3. **Select the model** (`LIFEOS/TOOLS/models.ts`) — The level from stage 2 resolved to a concrete model through `EFFORT_MODEL` — the one place the four levels bound to the lineup:
   - `max` → Fable (Algorithm E4/E5 + Core-System Override)
   - `high` → Opus (Algorithm E1–E3 + NATIVE delegated work, ~90% rung)
   - `medium` → Sonnet (utility inference: summaries, classification, vision triage)
   - `low` → Haiku (cheap lookups)
4. **Dispatch the agent** — The resolved model was passed as the `model` parameter on `Agent({…, model})`. Set at dispatch because static agent frontmatter could not be tier-conditional.

### The three level axes (don't conflate them)

Three independent dials shared the words "max" and "high":

- **Model rung** (`EFFORT_MODEL` in `models.ts`) — which model in the lineup
- **Harness reasoning effort** (`--effort` flag: `low`/`medium`/`high`/`xhigh`/`max`) — how hard the model thinks
- **Agent effort override** (`/effort` in the prompt) — per-agent override

The crossover that tripped people up: "max model" and "max thinking" were different dials. The single source of truth for the rung→effort crossover was `LEVEL_TO_HARNESS_EFFORT` in `models.ts`.

### The classifier contract

On every top-level prompt the classifier wrote these lines into `additionalContext`; the executor read them directly:

```
MODE: MINIMAL | NATIVE | ALGORITHM
TIER: E1 | E2 | E3 | E4 | E5 (only when MODE=ALGORITHM)
REASON: <one sentence>
SOURCE: classifier | fail-safe | fast-path | cache | explicit
GOAL_SIGNAL: 1 | 2 | 3 | 4 | none
GOAL_LITERAL: "<verbatim prompt quote>" (when a goal is detected)
INTERVIEW_ELIGIBLE: true | false
```

### Cross-vendor egress

The Router also picked the **vendor**, and each vendor route carried a data-sensitivity ceiling. The `models.ts` routing encoded a per-route ceiling (source + model + residency), most-restrictive-wins. See the **Security** skill for the full data-classification × inference-source routing matrix.

## Hermes-Native Mapping

The Router's function — deciding how to handle a prompt — is distributed across Hermes-native mechanisms:

### Model selection

| LifeOS Router | Hermes-native | Notes |
|---|---|---|
| `EFFORT_MODEL` four-level abstraction (`max`/`high`/`medium`/`low`) | `config.yaml` `model.default` + provider config | Hermes has a single default model per profile, not a four-level abstraction. The DA selects models through config, not runtime classification. |
| `CURRENT`/`ALIAS` model IDs | Provider model names in `config.yaml` | No alias layer. Provider and model are set directly. |
| `LEVEL_TO_HARNESS_EFFORT` crossover | Model-specific reasoning effort (provider-dependent) | Hermes does not expose a separate effort dial. The model's reasoning depth is determined by the model itself and provider parameters. |
| Cross-vendor pins | Provider configuration in `config.yaml` | Multiple providers can be configured. The DA selects which provider to use per task. |
| `Inference.ts` utility-path inference | Direct tool calls (`web_search`, `web_extract`, `vision_analyze`, etc.) | No separate utility-inference layer. Tools run directly. |

### Effort calibration

| LifeOS Router | Hermes-native | Notes |
|---|---|---|
| `MODE: MINIMAL` (reflexive answer, no phases) | DA judgment — direct response | The DA decides when a question is trivial and responds directly without loading skills or running the Algorithm. |
| `MODE: NATIVE` (tight template, delegated work) | DA judgment + `delegate_task` | The DA delegates work to subagents when appropriate. No mode banner. |
| `MODE: ALGORITHM` (seven phases, ISC gates) | Algorithm skill (`/skill Algorithm`) | The DA loads the Algorithm skill for substantial work. |
| `TIER: E1–E5` effort tiers | DA judgment + skill loading | No fixed tiers. The DA calibrates effort by choosing which skills to load, whether to delegate, and how many verification passes to run. |
| `/e1`–`/e5` explicit tier override | User direction + DA judgment | The user can explicitly direct effort level ("just do X" vs "do this thoroughly"). The DA responds accordingly. |

### Agent dispatch

| LifeOS Router | Hermes-native | Notes |
|---|---|---|
| `Agent({…, model})` per-dispatch model | `delegate_task(..., model=...)` | Hermes delegation supports per-dispatch model override. See the **Delegation** skill. |
| Dispatch profile (tier → agent + level) | `delegation.*` config in `config.yaml` | Delegation config sets provider, model, concurrency, and nesting depth. |
| `🤖 DISPATCH` telemetry | `delegate_task` lifecycle tracking | Hermes tracks delegation lifecycle natively. No JSONL telemetry. |
| `OpenRouter.ts` / `EgressClassGuard` | Provider config + constitution §8 | Data egress is governed by provider selection and the security protocol. See **Security** skill. |

### The dynamic range principle

The Router's core principle — **preserve dynamic range** — still applies in Hermes, but it is enforced through DA judgment rather than a classifier hook:

- **Trivial work:** respond directly. No skills, no delegation, no phases.
- **Moderate work:** load relevant skills, run targeted tool calls, verify results.
- **Substantial work:** load the Algorithm skill, create an ISA, delegate parallel subtasks, run multiple verification passes.
- **Cross-vendor work:** delegate to subagents with different models when the task benefits from diverse perspectives.

The DA calibrates by choosing what to load, what to delegate, and how many passes to run — not by consulting a tier table.

## What is NOT ported

| LifeOS component | Reason |
|---|---|
| `TheRouter.hook.ts` | Retired 2026-07-11. No Hermes equivalent for a `UserPromptSubmit` classifier hook. DA judgment replaces it. |
| Mode banners (`MINIMAL`/`NATIVE`/`ALGORITHM`) | No mode system in Hermes. The DA calibrates effort implicitly. |
| E1–E5 effort tiers | No fixed tier system. DA judgment + skill loading replaces tiers. |
| `EFFORT_MODEL` four-level abstraction | Hermes uses a single default model per profile. No level→rung lookup. |
| `additionalContext` classifier contract | No classifier output block. Context comes from Hindsight recall, LCM, skills, and the constitution. |
| `LIFEOS/TOOLS/models.ts` | No equivalent file. Provider/model config lives in `config.yaml`. |
| `LIFEOS/TOOLS/Inference.ts` | No utility-inference layer. Tools run directly. |
| Telemetry (`effort-router.jsonl`, `intelligence-routing.jsonl`) | No JSONL telemetry. LCM and session DB track execution natively. |

## Cross-references

- **Algorithm** skill — the seven-phase execution loop (replaces the Router's execution counterpart)
- **Delegation** skill — subagent dispatch and model-tier matching
- **Security** skill — data classification × inference-source routing (the Router's egress boundary)
- **Config** skill — Hermes config layering (replaces `models.ts` as the model selection point)
- **Constitution** §3 (Execution loop) and §6 (Layer boundaries)
- `PORT_SCHEMAS/hook_mapping.md` — `AgentInvocation.hook.ts` delegation model injection mapping

## Source

- `Router__RouterSystem` — the retired Router documentation (historical reference only)
