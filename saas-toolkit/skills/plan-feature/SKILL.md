---
name: plan-feature
description: Feature planning with phased spec output.
disable-model-invocation: true
argument-hint: name="<feature-name>"
allowed-tools:
  - Read
  - Grep
  - Glob
  - Task
  - WebSearch
  - WebFetch
  - mcp__context7
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

#### SaaS-specific design considerations

**Subscription gating:**
- Which subscription tier(s) can access this feature? (free / pro / enterprise)
- Is this a hard gate (blocked) or soft gate (limited usage)?
- Where is the subscription check enforced? (middleware, server action, component)
- What does the upgrade prompt look like?

**Multi-tenancy:**
- Is this feature user-scoped or organization-scoped?
- RLS implications: what policies are needed?
- Does the data model need `org_id` foreign keys?
- How does this interact with existing org member roles?

**Billing impact:**
- Does this feature change metered usage?
- Are new Stripe products/prices needed?
- Does the pricing page need updating?
- Any proration or plan change implications?

**Auth requirements:**
- Does this feature require authentication?
- What role/permission level is needed?
- Are there public (unauthenticated) views?
- OAuth or email verification requirements?

### 4. Implementation plan

Produce a phased plan with this structure:

```markdown
## Feature: [name]

### Overview
Brief description of what this feature does.

### Phase 1: [Foundation — DB & Types]
- [ ] Task 1 — file(s) affected, what changes
- [ ] Task 2 — ...

### Phase 2: [Core logic — Server Actions & API]
- [ ] Task 3 — ...
- [ ] Task 4 — ...

### Phase 3: [UI / Integration]
- [ ] Task 5 — ...

### Phase 4: [Polish / Edge cases]
- [ ] Task 6 — ...

### SaaS Feature Checklist
- [ ] RLS policies cover all CRUD operations
- [ ] Subscription tier gating implemented
- [ ] Audit logging for important actions
- [ ] Error states (auth, permission, subscription, network)
- [ ] Loading states (skeleton/spinner)
- [ ] Mobile responsive
- [ ] Empty states with helpful guidance
- [ ] Rate limiting (if applicable)

### Risks & open questions
- ...
```

## Rules

- Do NOT write code during planning — this is a planning-only skill
- Be specific about files and changes, not vague
- Consider error handling, loading states, and edge cases
- Flag decisions that need user input
- Keep the plan realistic and achievable
- Always address subscription gating and multi-tenancy for new features
