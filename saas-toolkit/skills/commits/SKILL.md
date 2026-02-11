---
name: commits
description: Create conventional commits with structured type, scope, and body.
disable-model-invocation: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# /commits — Conventional Commit Workflow

Create well-structured git commits following the Conventional Commits specification.

## Process

1. **Review changes** — Run `git status` and `git diff --staged` to understand what's being committed. If nothing is staged, run `git diff` to see unstaged changes and ask which files to stage.

2. **Determine commit type** — Based on the changes:
   - `feat` — new feature or functionality
   - `fix` — bug fix
   - `refactor` — code restructuring without behavior change
   - `style` — formatting, whitespace, missing semicolons
   - `docs` — documentation only
   - `test` — adding or updating tests
   - `chore` — build, tooling, dependencies, config
   - `perf` — performance improvement
   - `ci` — CI/CD configuration

3. **Determine scope** — Identify the affected area (e.g., `auth`, `billing`, `ui`, `api`, `db`). Use the directory or feature name.

4. **Write commit message** — Format:
   ```
   type(scope): short description

   Longer body explaining what changed and why, if needed.
   ```

5. **Stage and commit** — Stage the relevant files (prefer explicit file names over `git add .`) and create the commit.

## Rules

- Keep the subject line under 72 characters
- Use imperative mood ("add feature" not "added feature")
- Do not include file lists in the commit body — git tracks that
- Group related changes into a single commit
- Separate unrelated changes into multiple commits
- Never use `--no-verify` unless explicitly asked
