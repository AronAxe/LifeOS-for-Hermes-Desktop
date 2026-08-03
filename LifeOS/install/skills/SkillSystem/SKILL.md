---
name: SkillSystem
version: 1.0.0
description: "Authoritative definition and Hermes-native mapping for the LifeOS Custom Skill System. USE WHEN you need to know the required structure of a skill, naming conventions, public vs private boundaries, or how to map LifeOS skill concepts to the Hermes runtime."
effort: low
---

# SkillSystem

Skills are the LifeOS action surface. This skill defines the mandatory structure, naming conventions, and Hermes-native mappings for all skills. A skill that doesn’t follow this structure is not properly configured.

## Hermes-Native Mapping

LifeOS concepts adapt to Hermes as follows:

*   **Frontmatter & Management**: Use Hermes skill frontmatter. Manage via `skill_view`, `skills_list`, and `skill_manage` instead of Claude-specific file parsing.
*   **Preferences & Knowledge**: Use **Hindsight** for durable preferences and knowledge, not a local file-memory runtime.
*   **Invariants**: Rely on the **Hermes constitution** for invariants.
*   **Forbidden Assumptions**: No Claude `settings.json`, no `launchd`, and no local file-memory runtime assumptions.
*   **Provenance**: A LifeOS install skill is source/provenance material. A Hermes runtime skill must remain strictly generic unless it is explicitly marked private.

## Naming Convention — Public vs Private (MANDATORY)

A skill’s name encodes its public/private status:

| Skill type | Directory format | Example | Allowed content |
|------------|------------------|---------|-----------------|
| **Public** | `TitleCase` | `Blogging`, `Daemon` | Templated, safe, generic, ready for public release. |
| **Private** | `_ALLCAPS` | `_MYFINANCES`, `_HOMELAB` | Anything personal, identity-bound, customer-bound, or environment-specific. |

**The leading underscore is the public-release boundary.** Release tooling skips `_*` skills. Public skills MUST contain only generic, templated content.

### Sub-file Naming
*   Workflow files: `TitleCase.md` (e.g., `Create.md`)
*   Reference docs: `TitleCase.md` (e.g., `ApiReference.md`)
*   Tool files: `TitleCase.ts` (e.g., `ManageServer.ts`)

## Skill Structure & Dynamic Loading

Keep folder structure flat (maximum 2 levels deep).

**Allowed Subdirectories:**
*   `Workflows/` - Execution workflows ONLY.
*   `Tools/` - Executable scripts/tools ONLY.
*   Additional context files - stored directly in the skill root.

**Dynamic Loading Pattern:**
Keep `SKILL.md` minimal (routing + quick reference). Additional `.md` files are context files (SOPs) loaded on-demand, located directly in the skill root (NO `Context/` or `Docs/` subdirectories).

## Authoring Standard — Ideal-State Prompting

Write every new skill body and workflow ideal-state style: articulate WHAT a done deliverable looks like (as testable outcomes), the CONSTRAINTS, and the TOOLS — then trust the model to find HOW. Avoid BPE-violating numbered step-lists for cognitive work. Keep: safety-gates, verified-gotchas, tool-contracts, and output-format-contracts.

## Workflow Routing

This port documents the routing contract without claiming workflow files that are not present in this repository. When a skill has workflows, route by intent using this shape:

| Workflow | Trigger | File |
|----------|---------|------|
| **ValidateStructure** | "validate structure", "check skill format" | `Workflows/ValidateStructure.md` |
| **MapToHermes** | "map to hermes", "hermes concepts" | `Workflows/MapToHermes.md` |

For direct Hermes skill administration, use `skills_list`, `skill_view`, `skill_manage`, and `read_file`; do not infer that the illustrative files above exist.

## Examples

**Example 1: Determining Skill Privacy**
User: "Should my skill with client API keys be public or private?"
→ Invokes SkillSystem knowledge
→ Explains that real credentials force a private `_ALLCAPS` skill.
→ Recommends `_CLIENTWORK` naming.

**Example 2: Mapping a Claude Skill to Hermes**
User: "How do I map a Claude settings.json hook in my new skill?"
→ Invokes MapToHermes workflow
→ Explains that settings.json hooks are not used in Hermes.
→ Directs the user to use the Hermes constitution and Hindsight for durable memory.

## Gotchas
*   Do not invent a separate Hermes runtime; skills belong to the installed Hermes skill body.
*   Never use `SKILLCUSTOMIZATIONS` to smuggle private content into a public skill.
