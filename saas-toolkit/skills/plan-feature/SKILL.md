---
description: Feature planning with phased spec output
allowed-tools:
  - Read
  - Grep
  - Glob
  - Task
  - WebSearch
  - WebFetch
  - mcp__context7__*
user-invocable: true
---

# /plan-feature — Feature Planning

Create a detailed implementation plan for a new feature, producing a structured spec that can be handed off to `/dev`.

## Process

### 1. Requirements gathering

- Clarify the feature request with the user
- Identify acceptance criteria
- Determine scope boundaries (what's in, what's out)

### 2. Codebase analysis

- Explore the current codebase to understand existing patterns
- Identify files and modules that will be affected
- Check for existing utilities or components that can be reused
- Look for similar features already implemented

### 3. Technical design

- Choose the implementation approach
- Define the data model changes (if any)
- Plan the API surface (routes, server actions, etc.)
- Design the UI components (if applicable)
- Identify third-party integrations needed

### 4. Implementation plan

Produce a phased plan with this structure:

```markdown
## Feature: [name]

### Overview
Brief description of what this feature does.

### Phase 1: [Foundation]
- [ ] Task 1 — file(s) affected, what changes
- [ ] Task 2 — ...

### Phase 2: [Core logic]
- [ ] Task 3 — ...
- [ ] Task 4 — ...

### Phase 3: [UI / Integration]
- [ ] Task 5 — ...

### Phase 4: [Polish / Edge cases]
- [ ] Task 6 — ...

### Risks & open questions
- ...
```

## Rules

- Do NOT write code during planning — this is a planning-only skill
- Be specific about files and changes, not vague
- Consider error handling, loading states, and edge cases
- Flag decisions that need user input
- Keep the plan realistic and achievable
