---
description: Quick one-shot tasks — minimal overhead, just do it
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
user-invocable: true
---

# /oneshot — Quick One-Shot Tasks

For small, well-defined tasks that don't need a full workflow. Just do it.

## Behavior

1. **Read the request** — Understand what's being asked.
2. **Do it** — Make the change directly. No planning phase, no lengthy analysis.
3. **Verify** — Quick check that it works (type check, no obvious errors).
4. **Report** — Brief summary of what was done.

## When to use

- Renaming a variable or component
- Adding a simple utility function
- Updating a config value
- Adding an import or export
- Small CSS/styling tweaks
- Adding a type definition
- Updating copy/text content
- Simple file moves or renames

## Rules

- Read the file before editing it
- Keep changes minimal and focused
- If the task turns out to be more complex than expected, say so and suggest using `/dev` instead
- No over-engineering — don't add types, tests, or docs unless asked
- Don't ask for confirmation on trivial changes — just do it
