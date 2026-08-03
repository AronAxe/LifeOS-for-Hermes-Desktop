---
name: Tools
description: "Use when selecting a direct Hermes tool, command, or durable script versus promoting a contextual, judgment-heavy, repeatable workflow into a skill."
version: 1.0.0
effort: medium
---

# Tools — Direct Utility Doctrine

This skill defines the doctrine for **Hermes-native** tool use, utility mapping, and the decision boundary between simple tool execution and full skill promotion.

## 1. The Decision Rule

When addressing a problem, apply the following principle to decide whether to use a direct tool/utility or create a skill:

*   **Direct Execution:** If a task is a simple, deterministic utility, document it and execute it directly via built-in tools or CLI commands. Do not inflate it into a skill.
*   **Skill Promotion:** If the job requires contextual judgment, intelligent routing, or a complex, repeatable multi-step workflow, promote it to a dedicated Skill.

## 2. Historical Utility Mapping

Old LifeOS utilities map directly to native Hermes capabilities. Do not attempt to clone the old implementations.

| Old LifeOS Utility | Hermes-Native Counterpart | Notes |
| :--- | :--- | :--- |
| `Inference.ts` (Model levels) | Configured Hermes model/provider & native tools | No custom inference clone needed; rely on platform model configuration. |
| `MemoryRetriever` | Hindsight recall/reflect & LCM | Use native long-term memory for session context instead of custom vector retrieval. |
| `KnowledgeGraph` | Hindsight & Cognitive Graph | Hindsight: durable entities, contacts, general/domain knowledge, ordinary factual relationships, and durable learnings. Cognitive graph: only reviewed values, heuristics, tensions, assumptions, mental models, and projects. Arbitrary Knowledge Archive tags, wikilinks, and related notes do not enter mind.db merely because they are linked or graph-shaped. |
| Localhost Voice Server | `text_to_speech` & `send_message`/gateway | No direct port. Use native audio/message routing. |
| `Monitor` | Background Processes & Cronjobs | Use terminal background tasks with `notify_on_complete`/watch patterns. **No sleep loops.** |
| `Doctor` | Live Probes & Diagnostics | Run live Hermes/tool probes for advisory diagnostics. Do not treat a static capability cache as ground truth. |
| Arbol Utilities | `terminal`, `execute_code`, or durable scripts | Use `terminal` for one shell command; use `execute_code` for three or more tool calls with processing, branching, or loops; retain a durable script only for proven recurrence. |

## 3. CLI vs. MCP Boundary

Consistent with the `CliFirstArchitecture`:
*   **Consume internally via CLI/tools:** A capability this harness uses should be a deterministic command, script, or native Hermes tool where practical.
*   **Serve externally via MCP:** Use MCP only when exposing a capability to clients outside this harness. Do not build an MCP server merely to call a local utility.

## 4. Tool Lifecycle

When tackling a new problem requiring external interaction, follow this lifecycle:

1.  **Discover:** Search for existing CLI tools, native Hermes built-ins, or simple scripts that address the need.
2.  **Run:** Execute the tool in the most direct manner possible.
3.  **Read Back:** Parse and analyze the output directly.
4.  **Document:** Record only a generic reusable pattern in source-controlled documentation or an appropriate skill; do not turn temporary task state or private operational notes into Hindsight.
5.  **Promote (If Warranted):** Only if the pattern becomes a complex, judgment-heavy workflow should you formalize it into a Skill.

## 5. Observability and Safety

*   **Observability:** All tool invocations must be traceable. Rely on terminal session logs and native Hermes execution transcripts. Background tasks must log to predictable locations.
*   **Safety:** Always prefer the narrowest scoped tool. Validate commands before execution, particularly destructive operations. Do not construct raw shell strings with unsanitized inputs.

## 6. Examples

**Good:** Using `terminal` with a narrowly scoped search command, reading the output directly, and applying a verified fix.
**Bad:** Creating a complex "GrepSearchAndReplace" skill for a one-off refactor that could be handled by standard tools.

**Good:** Using the native scheduler/cron tools to run a background diagnostic script every hour.
**Bad:** Writing a custom Node.js script that uses `setInterval` and running it indefinitely in the background.

## 7. Common Pitfalls

*   **Premature Skill Abstraction:** Creating a skill for a three-line shell command.
*   **Reinventing the Wheel:** Building custom inference wrappers or memory databases instead of using native Hermes capabilities.
*   **Sleep Loops:** Using blocking `sleep` calls for polling instead of native background task management and watch APIs.
*   **Capability Caching:** Assuming a system state is valid based on an old check rather than live probing.

## 8. Verification Checklist

- [ ] Does the approach use the simplest possible tool?
- [ ] Are old LifeOS utilities correctly mapped to Hermes native counterparts without custom reimplementation?
- [ ] Is the CLI preferred over MCP for system-level tasks?
- [ ] Are background tasks managed via native notify/cron patterns instead of sleep loops?
- [ ] Has the capability been verified via live probe rather than a static cache?

## Cross-References

*   `CliFirstArchitecture`
*   `CLI`
*   `Testing`
*   `Observability`
*   `Security`
*   `constitution`
