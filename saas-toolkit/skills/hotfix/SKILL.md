---
description: Hotfix debugging workflow — reproduce, diagnose, fix, verify
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - mcp__supabase__*
user-invocable: true
---

# /hotfix — Hotfix Debugging Workflow

Structured workflow for quickly diagnosing and fixing bugs in production or development.

## Phases

### Phase 1: Reproduce

1. **Understand the bug** — Get a clear description of the expected vs actual behavior.
2. **Identify the trigger** — What user action or condition causes the bug?
3. **Check logs** — Look for error messages, stack traces, or relevant log output.
4. **Reproduce locally** — If possible, reproduce the issue in the development environment.

### Phase 2: Diagnose

1. **Trace the code path** — Follow the execution flow from the trigger point to the error.
2. **Identify the root cause** — Find the exact line or logic error causing the bug. Don't stop at symptoms.
3. **Check for related issues** — Is this a one-off bug or part of a pattern? Are there similar bugs elsewhere?
4. **Assess impact** — What's affected? Just this page? All users? Data integrity?

### Phase 3: Fix

1. **Minimal fix** — Fix the root cause with the smallest possible change. Don't refactor surrounding code.
2. **Handle edge cases** — If the bug revealed missing edge case handling, address it.
3. **Don't break other things** — Check that the fix doesn't introduce regressions.

### Phase 4: Verify

1. **Test the fix** — Verify the bug is resolved.
2. **Run existing tests** — Ensure nothing else is broken.
3. **Check related flows** — Test adjacent features that might be affected.
4. **Commit** — Use `fix(scope): description` commit format.

## Rules

- Focus on the root cause, not symptoms
- Keep fixes minimal and focused
- Don't refactor or improve code unrelated to the bug
- If the fix is complex, explain the reasoning
- If you can't find the root cause, say so — don't guess
