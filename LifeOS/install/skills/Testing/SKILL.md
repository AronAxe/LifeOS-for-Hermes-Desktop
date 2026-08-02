---
name: Testing
version: 1.0.0
description: "Hermes-native testing doctrine. Enforces evidence-first verification, where a claim is only done when a probe produces real evidence. Defines verification levels for various artifacts, anti-criteria rules, state-machine transitions, and test hermeticity."
effort: low
---

# Testing — Evidence-First Verification

## What It Does
Testing is how the system knows what it knows. This skill enforces the Hermes-native testing doctrine: every claim in an ISA must be verified by a tool probe, and an unverified claim is just a guess wearing a checkmark. It defines verification levels, rules for deterministic and isolated tests, and how to map tests to the repository's native layout.

## Doctrine Rules (Non-negotiable)

1. **Evidence-First Verification**: A claim is not done until a probe produces real evidence. The ISA `## Test Strategy` links criteria directly to executable tools. Never invent a green result.
2. **Repository-Native Testing**: Inspect the actual repository and runtime. Use the native testing frameworks and direct execution appropriate to the artifact. Keep tests in the repository's established layout rather than assuming a parallel tree. *Note: The source doctrine's Bun harness is NOT a Hermes runtime dependency.*
3. **Anti-Criteria are First-Class**: Ensure regressions do not happen. If an ISA declares an `Anti:` criterion, the test must explicitly prove the unwanted behavior does not occur.
4. **State-Machine Transitions**: A state-machine test must *produce* the transition (e.g., crossing from open to closed). Do not assert behavior by handwriting the state into a fixture.
5. **No Flakiness, No Retries**: No retries, no time-based sleeps, no hardcoded ports, and no per-test timeouts. Tests must be deterministic. Normalize nondeterminism (like timestamps or paths) before asserting.
6. **Disposable Fixtures**: Use disposable scopes for anything mutable (temporary directories, environments, configurations).
7. **No Billed External Inference**: Tests must not invoke actual, billed external LLM inference. Mock the provider or replay fixture responses.
8. **Property Testing is Optional**: When property testing is used, use the repository's available approved primitive rather than assuming Bun dependencies.
9. **Long-Horizon State is External to Memory**: If work crosses context windows, keep a small machine-readable test state file with the workspace/ISA artifacts; do not put test checklist state into Hindsight.

## Verification Levels

Match the verification probe to the artifact being tested:

| Artifact | Verification Approach |
|----------|------------------------|
| **Markdown Skills** | Read-back the generated or updated file with `read_file`; ensure valid frontmatter and required section structure. |
| **Scripts & Code** | Direct execution with checked exit codes and stdout/stderr output. Native test runners if available. |
| **Configuration** | Read-back of the applied config. |
| **Memory Operations** | A successful provider result plus a recall/read-back check to confirm persistence. |
| **Remote Changes** | Live probe of the deployed URL or ID, along with a read-back of the remote state. |

## Examples

### State-Machine Transition
```javascript
// WRONG: Manually setting state to assert outcome
state.isClosed = true;
expect(process()).toBe(false);

// RIGHT: Producing the transition
process(); // open
closeProcess(); // transition
expect(process()).toBe(false); // verification
```

### Anti-Criterion Verification
```javascript
test("Anti: API Key is not leaked in output", () => {
    const result = runCommand();
    expect(result).not.toContain("API_KEY");
});
```

## Cross-References
- **Algorithm**: Defines the VERIFY phase where this doctrine is executed (`/skill Algorithm`).
- **ISA**: The Ideal State Artifact that holds the claims and `## Test Strategy` (`/skill ISA`).
- **Security**: Ensures no sensitive credentials leak in tests (`/skill Security`).
- **Observability**: Exposes test results and system state (`/skill Observability`).
- **Constitution**: Implements verification and honesty invariants from `HERMES_CONSTITUTION.md`.
