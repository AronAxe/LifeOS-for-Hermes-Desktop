---
name: Containment
version: 1.0.0
description: "Use when creating, modifying, or reviewing portable skills, configuration, or documentation. Prevents personal data, credentials, private infrastructure identifiers, and user-specific paths from entering releasable artifacts."
effort: medium
---

# Containment Policy

**Core Rule:** Portable repository content must be strictly clean of personal identity, credentials, private infrastructure identifiers, and absolute user-specific paths. **The OS ships; the life does not.**

This skill defines the boundaries between portable, public system components and private, instance-specific data within the Hermes environment. Conceptualizing containment zones ensures that what is meant to be shared is safe, and what is meant to be private remains secure.

## Public vs. Private Boundaries

### Public / Releasable Scope
These artifacts are designed to be shared, deployed, or committed to public repositories:
- Generic `TitleCase` skill directories (e.g., `Containment`, `Planning`).
- Portable `install/` documentation and architectural schemas.
- Generic automation scripts and templates using placeholders.

### Private / Local Scope (Do Not Ship)
These artifacts contain instance-specific data and must remain strictly local and outside the release scope:
- Profile-scoped configuration files.
- `.env` files, secrets, and API keys.
- Private `_underscore` skills (e.g., `_my_private_skill`).
- Local memory, session data, and caches.
- Private infrastructure details (IP addresses, specific cloud bucket names).
- **Hindsight:** Hindsight is durable personal data and inherently does not belong in portable artifacts.

- **Containment is a continuous review rule:** Hermes does not reproduce the source system's static zone inventory, release gates, or write-time Claude hook. The portable/private boundary is enforced through explicit scope review, disciplined staging, and the constitution's safety constraints.

## Configuration and Placeholders

Whenever a configurable value, path, or identifier is required in a public artifact, you must use generic placeholders or environment variables.
- **Incorrect:** A literal personal home-directory path or a real personal email address.
- **Correct:** `${HOME}/projects/data` or `<USER_EMAIL>`.

## Pre-Commit and Release Review

Before committing or releasing any artifact, an explicit review must be conducted:

1. **Inspect and Stage Deliberately:** Review `git status` and `git diff`. Never use `git add .` blindly; stage changes file-by-file.
2. **Scan for Leaks:** Manually or automatically scan changed public artifacts for secrets, private paths, and personal identifiers.
3. **No Casual Exceptions:** Do not bless new exceptions casually. The containment boundary is rigid.

### Exceptions
An exception to the containment rule is allowed *only* if a file must explicitly contain a detection pattern (e.g., a regex designed to find absolute paths or a literal string for a test). In such cases:
- Document exactly why the pattern or exception is there in comments.
- Minimize the scope of the exception to only what is strictly necessary.

## Common Situations & Pitfalls

- **Pitfall - Copy-Pasting Shell Output:** Pasting an error message or terminal output into a public documentation file might accidentally include your absolute path (e.g., `C:/Users/.../`). Always scrub pasted output.
- **Situation - Creating a New Skill:** When scaffolding a new skill, ensure the directory name is generic `TitleCase`. If the skill requires API keys, document that the user should create an `.env` file, but do not create one in the public template.
- **Pitfall - Hardcoded Infrastructure:** Hardcoding an internal local IP or specific server name in a deployment script. Use `<TARGET_SERVER_IP>` or an environment variable like `$DEPLOY_HOST` instead.

## Example: Release-Safe Configuration

A portable deployment template needs a target host and an operator email. It must take them from `<TARGET_SERVER_IP>`, `<USER_EMAIL>`, or documented environment variables; it must not embed a real hostname, email address, home directory, token, or Hindsight-derived note. If a scanner must include a representative detection pattern, confine that pattern to the scanner/test file, explain why it cannot be genericized, and review it as a narrow exception.

## Verification Checklist

When working on releasable artifacts, verify the following before completion:
- [ ] No absolute user paths (for example, a real home-directory path).
- [ ] No personal emails, names, or usernames.
- [ ] No API keys, passwords, or tokens.
- [ ] No private infrastructure identifiers (internal IPs, specific cloud project IDs).
- [ ] Configurable values use generic placeholders or environment variables.
- [ ] `git diff` has been reviewed line-by-line for the current staging area.
- [ ] Hindsight data, local memory, and caches are excluded from the commit/release.
