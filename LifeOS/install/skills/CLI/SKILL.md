---
name: CLI
version: 1.0.0
description: "Use when designing, executing, or composing deterministic command-line work in Hermes. Maps retired LifeOS Arbol CLI concepts to terminal, execute_code, scripts, and native scheduling without recreating a custom runner."
effort: low
---

# CLI — Deterministic Command Composition

## Overview

This document adapts the LifeOS Command-Line Tools and Arbol CLI doctrine into a Hermes-native model. A command is the execution surface for a stable operation: it accepts explicit input, produces inspectable output, and can be exercised without a prompt. The prompt decides **which** command to compose; the command owns **how** the operation executes.

## When to Use

Use this skill when designing a command interface, composing JSON-compatible command output, deciding between `terminal`, `execute_code`, a durable script, or a higher-level skill, or translating a legacy action/pipeline workflow. Do not use it for a one-off question with no deterministic operation to preserve.

## Historical Context & Boundary

The legacy LifeOS CLI tools—specifically the private Arbol CLI (`pai`), its custom runner, and the standalone Algorithm CLI—are **not** Hermes runtime dependencies. The standalone Algorithm runner is officially retired.

## The Core Model (Preserved Principles)

Rather than copying the legacy runtime, we preserve its functional model for command-line tools:

1. **Deterministic Operations**: Actions are deterministic with explicit, predictable input and output.
2. **Data Pipelines**: Pipelines compose data using standard, JSON-compatible formats.
3. **Flexible Inputs**: Standard input, explicit arguments, and named flags are used to support both interactive use and automated scripting.
4. **Stream Separation**: Operational diagnostics, logs, and errors must be routed to `stderr`. Pipeline data and meaningful output must stay on `stdout`.

## Command Contract

A reusable command or script should have an explicit, documented contract:

- Support `--help`; use hierarchical verbs when the capability has multiple operations.
- Accept a documented input precedence across standard input, explicit structured input, and named flags.
- Keep structured/pipeline data on stdout; emit progress, diagnostics, and errors on stderr.
- Offer human-readable output by default and a JSON/machine-readable mode when scripts consume the result.
- Be idempotent where safe; reserve `--force` for deliberately bypassing a guard.
- Provide `--dry-run` or `--explain` before consequential mutations.
- Exit `0` on success and a documented non-zero code on failure. Error messages must state the failed input and the next corrective action.
- Verify the command directly before a prompt begins orchestrating it.

## Mapping to Hermes

Legacy concepts (pai/action/pipeline) map directly to native Hermes capabilities:

- **Pipelines & Actions:** Use `terminal` for one direct shell command with a clear, bounded result.
- **Complex Multi-Call Processing:** Use `execute_code` when three or more tool calls need data processing, branching, filtering, or looping between them; it is an orchestrator, not a new CLI runtime.
- **Persistence & Automation:** Promote proven recurrence to a durable script, then invoke it through `terminal`, a tracked background process, or `cronjob` as appropriate.

### Inference & Models

Model selection and inference are handled entirely by the configured Hermes model/provider and native tool-calling features. Do **not** attempt to port legacy LifeOS model levels or create custom model wrappers.

## Usage Guidelines

- **When to use a script/command**: For deterministic, repeatable data processing or automation where the inputs and outputs are clearly defined.
- **When to use a skill**: For judgmental, multi-step work requiring context evaluation and adaptive decision-making.
- **When NOT to construct a custom runner**: Do not build bespoke CLI runners or execution environments; rely on the Hermes terminal and built-in capabilities.

## Examples

### Pipelining Data (Generic)
```sh
# Generate data, filter it, and process the results
generate_data --format json | filter_tool --query "status=active" | process_tool --output result.json
```

### Diagnostic vs. Data Output
```sh
# process_tool outputs progress to stderr and valid JSON to stdout
cat input.txt | process_tool > output.json 2> logs.txt
```

## Common Pitfalls

- **Recreating pai:** Do not build a bespoke runner merely to provide a wrapper around native Hermes execution.
- **Polluting stdout:** JSON and text pipelines fail quietly when progress logs share the data stream.
- **Hidden model routing:** Do not recreate retired LifeOS inference levels; use the configured Hermes provider/model and direct native tools.
- **Prompt-owned plumbing:** A repeated chain of ad-hoc commands is a signal to create and verify a script.
- **Unverified automation:** A script that has not been exercised directly is not a capability; it is a hypothesis with a filename.

## Verification Checklist

- [ ] The command executes non-interactively from a documented input shape.
- [ ] `--help`, errors, exit status, output format, and mutation behavior have been checked.
- [ ] Structured data remains parseable on stdout while diagnostics remain on stderr.
- [ ] The command works directly before any prompt or scheduled workflow wraps it.
- [ ] No legacy Arbol, Bun, Claude, macOS, or retired Algorithm-runner dependency has leaked into the Hermes mapping.

## Cross-References

- CliFirstArchitecture
- Tools
- Testing
- BackgroundServices
- HERMES_CONSTITUTION.md
