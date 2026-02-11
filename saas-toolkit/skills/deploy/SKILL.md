---
name: deploy
description: Deploy to Vercel. Use when setting up deployment, configuring env vars, or troubleshooting builds.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Task
---

# /deploy — Deployment Preparation

Pre-deployment checklist and readiness verification for Vercel deployment of Next.js + Supabase + Stripe SaaS apps.

## Process

### 1. Build verification

```bash
npm run build
```

Check for:
- Build completes without errors
- No TypeScript errors
- No ESLint errors (if configured)
- Build output size is reasonable

### 2. Environment variables audit

Verify all required env vars are documented and set in Vercel:

**Required variables:**
| Variable | Where | Purpose |
|----------|-------|---------|
| `NEXT_PUBLIC_SUPABASE_URL` | Public | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Public | Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | Server only | Supabase admin access |
| `STRIPE_SECRET_KEY` | Server only | Stripe API |
| `STRIPE_WEBHOOK_SECRET` | Server only | Webhook verification |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Public | Stripe client |
| `NEXT_PUBLIC_SITE_URL` | Public | App URL for redirects |

**Check:**
- No `NEXT_PUBLIC_` on server-only secrets
- No hardcoded values in source code
- `.env.local` not committed to git
- `.env.example` file exists with all required vars (no values)

### 3. Database readiness

- All migrations applied to production Supabase
- RLS enabled on all public tables
- Indexes exist for foreign keys and frequently queried columns
- Seed data NOT applied to production
- Database types are up to date

### 4. Stripe readiness

- Webhook endpoint URL set in Stripe Dashboard for production
- Webhook secret is the production secret (not test/CLI secret)
- Products and prices created in live mode
- Customer Portal configured
- Stripe API version matches code expectations

### 5. Security checks

Launch the **security-reviewer** agent for a quick scan, or reference recent `/security-scan` results. Key checks:

- No secrets in source code
- RLS on all tables
- Auth middleware configured
- Webhook signature verification in place
- CORS and security headers configured

### 6. Performance checks

- Images use `next/image` with proper sizing
- Fonts use `next/font`
- Large components use `next/dynamic` for code splitting
- No unnecessary `"use client"` directives
- Suspense boundaries for streaming

### 7. Vercel configuration

Check `vercel.json` (if exists) and `next.config.js`:
- Framework preset: Next.js
- Node.js version compatible
- Build command and output directory correct
- Redirects and rewrites configured
- Headers (security headers, CORS) configured

## Output: Deployment Readiness Report

```markdown
## Deployment Readiness Report

**Date:** YYYY-MM-DD
**Status:** READY / NOT READY

### Checklist

| Check | Status | Notes |
|-------|--------|-------|
| Build | PASS/FAIL | ... |
| TypeScript | PASS/FAIL | ... |
| Env vars | PASS/FAIL | ... |
| Database | PASS/FAIL | ... |
| Stripe | PASS/FAIL | ... |
| Security | PASS/FAIL | ... |
| Performance | PASS/FAIL | ... |
| Vercel config | PASS/FAIL | ... |

### Blocking Issues
1. ...

### Warnings
1. ...

### Deploy Command
When ready:
- Push to main branch for auto-deploy, or
- `vercel --prod` for manual deploy
```

## Rules

- Do NOT deploy — only verify readiness
- Do NOT modify files — report issues for the user to fix
- Flag any blocking issue that would cause production failures
- Distinguish between blockers and warnings
- Be thorough — a missed issue in production is worse than a delayed deploy
