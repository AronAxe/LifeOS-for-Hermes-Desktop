---
name: CliFirstArchitecture
version: 1.0.0
description: "Use when designing a new deterministic capability, deciding between a CLI/tool and a prompt, composing scripts, or choosing the internal CLI versus external MCP boundary."
effort: medium
---

# CliFirstArchitecture

## Overview

The CLI-First Architecture doctrine dictates that **deterministic executable operations precede prompts**. Prompts are designed to interpret intent and orchestrate capabilities, but the capabilities themselves must be fundamentally deterministic, inspectable, and repeatable.

By building a deterministic CLI or tool first, you establish a reliable foundation. The prompt then wraps this capability, rather than attempting to internally simulate or hardcode the execution logic.

## When to Use

**Use a CLI/Tool (Deterministic Operation) when:**
- The operation is repeated frequently.
- The outcome must be deterministic and verifiable.
- The execution needs to be inspectable (e.g., logging, error tracing).
- The task requires system state changes or complex local execution.

**Use a Prompt (One-Off Question) when:**
- The user has a genuine one-off question.
- Interpreting intent, natural language, or unstructured data is the primary goal.
- Orchestrating existing tools to achieve a novel outcome.

## Design Standards

A clean command surface is essential for AI-driven orchestration. Every tool or CLI must adhere to these standards:

- **Hierarchical Verbs:** Structure commands logically (e.g., `resource verb`, `config set`).
- **Help Documentation:** Always provide a `--help` flag with clear usage instructions.
- **Output Formatting:** Provide human-readable defaults for users, but include a `--json` or similar flag for machine-readable output.
- **Stream Separation:** Send clean pipeline data to `stdout` and diagnostic/error messages to `stderr`.
- **Error Handling:** Emit explicit non-zero exit codes on failure.
- **Idempotency:** Operations should be safe to retry. Use `--force` to override safety checks when necessary.
- **Safety Flags:** Support `--dry-run` or `--explain` to preview actions without side effects.
- **Configuration Flags:** Allow overriding defaults via explicit configuration flags.
- **Composability:** Design tools to be piped and chained together.
- **Verification:** Always verify the deterministic capability works independently before prompting wraps the capability.

## Hermes Mapping

When building capabilities within Hermes, map your requirements to the appropriate execution primitive:

- **Direct Terminal Commands:** Use for single, straightforward command executions.
- **Execute Code:** Use when a workflow requires 3+ calls involving data processing, branching, or looping logic that cannot be elegantly handled in a single command.
- **Scripts:** Use for durable, repeatable operations that will be invoked multiple times across sessions.
- **Cron / Background Processes:** Use for scheduled, asynchronous, or continuous background work.
- **Tools / MCP:** Use native Hermes tools or external MCP integrations when the capability is already provided. For a capability this harness consumes, prefer a deterministic CLI/script; use MCP only when serving the capability to an external client.

## The MCP Boundary

Preserve the strict MCP boundary doctrine:
- **Consume Internally:** Access capabilities internally via deterministic CLIs, scripts, or direct tool execution.
- **Serve Externally:** Only expose capabilities via MCP when serving an external client or extending capabilities beyond the local Hermes harness.

## Common Pitfalls

- **Prompting for Execution:** Asking the model to repeatedly recreate shell logic instead of first making the operation executable, tested, and reusable.
- **Mixing Output Streams:** Spewing logs to `stdout` alongside JSON data, breaking downstream parsers.
- **Lack of Idempotency:** Creating scripts that corrupt state if run twice.
- **Over-Engineering MCP:** Spinning up an MCP server for a local script that could just be executed directly.
- **Assuming Human Operators:** Forgetting `--json` or zero-interaction modes, causing the orchestration layer to get stuck on interactive prompts.

## Examples

### Good: Deterministic Script Orchestrated by Prompt
The user asks to clean up old logs. Hermes executes a deterministic script:
`cleanup-logs --days 30 --json`
Hermes reads the JSON output from `stdout` and summarizes the results for the user.

### Bad: Prompt Trying to be a Script
The user asks to clean up old logs. Hermes attempts to `find`, `grep`, `xargs`, and `rm` manually in the terminal, failing on edge cases like spaces in filenames, and cannot reliably parse the outcome.

## Verification Checklist

Before finalizing a new capability, verify:
- [ ] Is the core operation deterministic?
- [ ] Can it run without human interaction?
- [ ] Does it have a `--help` flag?
- [ ] Does it separate `stdout` (data) and `stderr` (logs/errors)?
- [ ] Are exit codes strictly enforced (0 for success, non-zero for failure)?
- [ ] Does it support idempotency or a `--dry-run` mode?
- [ ] Has the capability been tested independently of the prompt?

## Cross-References

- **CLI** — command and pipeline adaptation.
- **Tools** — selecting a direct utility rather than an unnecessary skill.
- **Testing** — evidence-first probe design.
- **BackgroundServices** — scheduled or long-running work.
- **Security** and **Containment** — egress, inputs, and portable-release boundaries.
- **HERMES_CONSTITUTION.md** — runtime and safety invariants.
