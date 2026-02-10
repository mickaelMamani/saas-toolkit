# Supabase + Next.js Reference Index

Compressed reference for Supabase integration with Next.js App Router.

## @supabase/ssr Client Patterns

### Server client (Server Components, Server Actions, Route Handlers)

```typescript
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export async function createClient() {
  const cookieStore = await cookies();
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return cookieStore.getAll(); },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          );
        },
      },
    }
  );
}
```

### Browser client (Client Components)

```typescript
import { createBrowserClient } from '@supabase/ssr';

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}
```

### Middleware client

```typescript
import { createServerClient } from '@supabase/ssr';
import { NextResponse, type NextRequest } from 'next/server';

export async function updateSession(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request });
  const supabase = createServerClient(URL, ANON_KEY, {
    cookies: {
      getAll() { return request.cookies.getAll(); },
      setAll(cookiesToSet) {
        cookiesToSet.forEach(({ name, value }) => request.cookies.set(name, value));
        supabaseResponse = NextResponse.next({ request });
        cookiesToSet.forEach(({ name, value, options }) =>
          supabaseResponse.cookies.set(name, value, options)
        );
      },
    },
  });
  await supabase.auth.getUser(); // Refresh session
  return supabaseResponse;
}
```

## Auth Flow

### PKCE Flow (default)
1. User clicks login → `supabase.auth.signInWithOAuth({ provider: 'github' })`
2. User redirected to provider → authenticates
3. Provider redirects to `/auth/callback?code=xxx`
4. Callback route exchanges code: `supabase.auth.exchangeCodeForSession(code)`
5. Session cookies set automatically

### Key auth methods

| Method | Use case |
|--------|----------|
| `signUp({ email, password })` | Email/password registration |
| `signInWithPassword({ email, password })` | Email/password login |
| `signInWithOAuth({ provider })` | OAuth login (GitHub, Google, etc.) |
| `signOut()` | Logout |
| `getUser()` | Get authenticated user (validates JWT) |
| `getSession()` | Get session (reads JWT only — NOT for auth checks) |
| `resetPasswordForEmail(email, { redirectTo })` | Password reset |
| `exchangeCodeForSession(code)` | PKCE callback |

### Session refresh
- Middleware calls `getUser()` on every request → auto-refreshes session
- No manual refresh needed

## Middleware Session Management

```typescript
// middleware.ts
import { updateSession } from '@/lib/supabase/middleware';
import type { NextRequest } from 'next/server';

export async function middleware(request: NextRequest) {
  return await updateSession(request);
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)'],
};
```

## RLS Policy Syntax

```sql
-- Enable RLS
ALTER TABLE public.my_table ENABLE ROW LEVEL SECURITY;

-- Policy syntax
CREATE POLICY "policy_name" ON public.my_table
  FOR [SELECT | INSERT | UPDATE | DELETE | ALL]
  [TO role_name]                              -- Optional: default is public
  [USING (condition)]                          -- Row visibility (SELECT, UPDATE, DELETE)
  [WITH CHECK (condition)];                    -- Row validity (INSERT, UPDATE)

-- Key functions
auth.uid()          -- Current user's UUID
auth.jwt()          -- Full JWT claims
auth.role()         -- Current role ('authenticated', 'anon')
```

### Common patterns

| Pattern | USING clause |
|---------|-------------|
| Owner access | `auth.uid() = user_id` |
| Org member | `auth.uid() IN (SELECT user_id FROM org_members WHERE org_id = table.org_id)` |
| Admin only | `...AND role = 'admin'` added to org member query |
| Public read | `true` |
| Authenticated only | `auth.role() = 'authenticated'` |

## Type Generation

```bash
# From local DB
npx supabase gen types typescript --local > lib/database.types.ts

# From remote
npx supabase gen types typescript --project-id <id> > lib/database.types.ts
```

Usage:
```typescript
import { Database } from '@/lib/database.types';

type Profile = Database['public']['Tables']['profiles']['Row'];
type InsertProfile = Database['public']['Tables']['profiles']['Insert'];
type UpdateProfile = Database['public']['Tables']['profiles']['Update'];
```

## Realtime

```typescript
const channel = supabase
  .channel('room-1')
  .on('postgres_changes', {
    event: '*',         // 'INSERT' | 'UPDATE' | 'DELETE' | '*'
    schema: 'public',
    table: 'messages',
    filter: 'room_id=eq.123',
  }, (payload) => {
    console.log('Change:', payload);
  })
  .subscribe();

// Cleanup
supabase.removeChannel(channel);
```

## Storage

```typescript
// Upload
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${userId}/avatar.png`, file);

// Download (public bucket)
const { data: { publicUrl } } = supabase.storage
  .from('avatars')
  .getPublicUrl('path/to/file');

// Download (private bucket)
const { data: { signedUrl } } = await supabase.storage
  .from('documents')
  .createSignedUrl('path/to/file', 3600);
```

## Edge Functions

```typescript
// supabase/functions/my-function/index.ts
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

Deno.serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );
  // Function logic
  return new Response(JSON.stringify({ ok: true }));
});
```

Deploy: `npx supabase functions deploy my-function`
