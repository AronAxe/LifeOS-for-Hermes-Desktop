---
name: Upgrade
version: 1.1.12
description: "Evaluates and recommends system improvements through an evidence-first lifecycle. Establishes current state, classifies prior status, and proposes testable recommendations for human approval. USE WHEN upgrade system, evaluate new feature, system improvement, propose upgrade, algorithm upgrade."
effort: high
---

# Upgrade

Evaluates and recommends system improvements through an evidence-first lifecycle: **observe → classify → recommend → human decision → implement → verify → learn**.

## Workflow

1. **Observe**: Establish actual current state using repository/config/workspace evidence (`read_file`, `search_files`, `terminal`) and `hindsight_recall` of past decisions. For external facts, use `web_search`/`web_extract` to gather sources.
2. **Classify**: Explicitly distinguish three candidate statuses before recommending:
   - *Already implemented*
   - *Rejected/contraindicated*
   - *Unverified candidate*
   Already implemented and rejected items are recorded as skipped with evidence and must not be re-pitched.
3. **Recommend**: For unverified candidates, formulate a recommendation that includes: evidence of current state, expected benefit, risk/reversibility, verification approach, and the minimal proposed change. Recommendations are proposals, not automatic mutations.
4. **Human Decision**: Recommendations require human approval. Unaccepted candidates remain ephemeral active workspace/ISA/review material; they are never durable Hindsight facts.
5. **Implement**: Approved implementations follow the Algorithm and real validation. Use only actual Hermes-native concepts/tools (e.g., workspace/ISA, optionally cognitive graph, `delegate_task` for genuinely independent work, and opt-in `cronjob` for recurring review).
6. **Verify**: Confirm the implemented change using tool evidence.
7. **Learn**: Only an approved durable decision, rule, or learning is written with `hindsight_retain` after confirmation; `hindsight_reflect` may be used to synthesize memory when useful but does not itself retain anything.

## Excluded Capabilities

Private source ledgers, UI surfaces, platform-specific hooks, notification endpoints, custom runners, private identifiers, automatic mutation, and raw telemetry/event logs are not Hermes runtime dependencies. Do not invent an upgrade queue, dashboard, API, notification system, or automated upgrade service.

## Gotchas

- Verify prior state before resurfacing an idea.
- Proposals are not approvals.
- Do not put live candidate state in Hindsight.
- Distinguish an upgrade candidate (system improvement) from a normal bugfix or current task.

## Cross-References

- **Algorithm**: For execution and validation of approved changes.
- **Memory/Hindsight Schema**: For durable retention of learnings and recall of past decisions.
- **ISA/Workspace**: For active task state and ephemeral review material.
- **Containment/Constitution**: For portable execution boundaries and safety.
- **Testing**: For verifying proposed changes before they are finalized.
