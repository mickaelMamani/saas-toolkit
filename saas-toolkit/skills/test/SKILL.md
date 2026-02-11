---
name: test
description: Write or run tests. Use when implementing new tests, fixing broken tests, or running the test suite.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash(npx vitest*, npx playwright*)
  - Task
---

# /test — Testing Workflows

Write, run, and analyze tests for SaaS applications using Vitest and Playwright.

## Modes

Specify which mode when invoking:
- `/test write` — Write tests for specified files/features
- `/test run` — Run existing tests
- `/test coverage` — Run tests with coverage analysis

### Mode: Write Tests

1. **Identify what to test** — Read the source file(s) and understand the behavior
2. **Check existing tests** — Look for existing test patterns in the project
3. **Write tests** following these patterns:

**Server Actions:**
```typescript
import { describe, it, expect, vi } from 'vitest';

// Mock Supabase
vi.mock('@/lib/supabase/server', () => ({
  createClient: vi.fn(() => mockSupabaseClient),
}));

describe('createProject', () => {
  it('creates a project for authenticated user', async () => {
    // Arrange: set up mocks
    // Act: call the server action
    // Assert: verify behavior
  });

  it('redirects to login if not authenticated', async () => { ... });
  it('returns error for invalid input', async () => { ... });
});
```

**API Routes (webhook handlers):**
```typescript
describe('POST /api/webhooks/stripe', () => {
  it('verifies webhook signature', async () => { ... });
  it('handles checkout.session.completed', async () => { ... });
  it('returns 400 for invalid signature', async () => { ... });
});
```

**React Components:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('PricingCard', () => {
  it('renders plan name and price', () => { ... });
  it('calls checkout on button click', async () => { ... });
  it('shows current plan badge for active subscription', () => { ... });
});
```

**Stripe integration tests:**
- Use Stripe test mode keys
- Mock `stripe.webhooks.constructEvent` for webhook tests
- Test subscription gating logic with different subscription states

### Mode: Run Tests

```bash
# Run all unit/integration tests
npx vitest run

# Run specific test file
npx vitest run path/to/test.ts

# Run tests matching pattern
npx vitest run -t "pattern"

# Run E2E tests
npx playwright test

# Run specific E2E test
npx playwright test path/to/test.ts
```

Report results to the user with pass/fail summary and any error details.

### Mode: Coverage

```bash
npx vitest run --coverage
```

Analyze coverage output and report:
- Overall coverage percentage
- Files with lowest coverage
- Untested critical paths (auth, billing, data access)
- Recommendations for where to add tests

## Test File Naming

- Unit/integration: `*.test.ts` or `*.test.tsx` next to source file
- E2E: `e2e/*.spec.ts`

## Rules

- Only run tests with `npx vitest` or `npx playwright` — no other bash commands
- Follow existing test patterns in the project
- Test behavior, not implementation details
- Keep tests independent — no shared mutable state
- Use descriptive test names: `it('redirects to login when not authenticated')`
- Launch the **testing-specialist** agent for complex test writing tasks
