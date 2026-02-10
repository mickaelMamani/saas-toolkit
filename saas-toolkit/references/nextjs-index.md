# Next.js App Router Reference Index

Compressed reference for Next.js 15 App Router patterns.

## File Conventions

| File | Purpose |
|------|---------|
| `layout.tsx` | Shared UI, persists across navigations. Receives `{ children }`. |
| `page.tsx` | Unique UI for a route. The only file that makes a route publicly accessible. |
| `loading.tsx` | Loading UI with Suspense. Wraps `page.tsx` automatically. |
| `error.tsx` | Error boundary. Must be `"use client"`. Receives `{ error, reset }`. |
| `not-found.tsx` | 404 UI. Triggered by `notFound()`. |
| `route.ts` | API endpoint. Exports `GET`, `POST`, `PUT`, `PATCH`, `DELETE`. |
| `template.tsx` | Like layout but re-mounts on navigation. |
| `default.tsx` | Fallback for parallel routes. |
| `middleware.ts` | Runs before every request (project root, not in `app/`). |

## Server Components vs Client Components

| | Server Component | Client Component |
|---|---|---|
| Default? | Yes | No — requires `"use client"` |
| Can fetch data | Yes (`async/await`) | No (use Server Component or Server Action) |
| Can use hooks | No | Yes |
| Can use event handlers | No | Yes |
| Can access browser APIs | No | Yes |
| Can import Server Component | Yes | No (pass as `children`) |
| Bundle size impact | Zero (not in JS bundle) | Added to JS bundle |

**Decision tree:** Does the component need hooks, event handlers, or browser APIs? → Client Component. Otherwise → Server Component.

## Server Actions

```typescript
// Inline in Server Component
async function handleSubmit(formData: FormData) {
  'use server';
  // mutation logic
}

// Separate file
// app/actions.ts
'use server';
export async function createItem(formData: FormData) { ... }
```

- Used for mutations (create, update, delete)
- Built-in CSRF protection
- Can be called from Client Components via `action` prop or direct call
- Revalidate data after mutation: `revalidatePath()` or `revalidateTag()`

## Data Fetching Patterns

**Server Component fetch:**
```typescript
// Direct async/await in component
export default async function Page() {
  const data = await fetchData(); // No useEffect needed
  return <div>{data.title}</div>;
}
```

**Parallel fetching:**
```typescript
const [users, posts] = await Promise.all([fetchUsers(), fetchPosts()]);
```

**Streaming with Suspense:**
```typescript
export default function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <AsyncComponent />
    </Suspense>
  );
}
```

**Caching:**
```typescript
// Route segment config
export const revalidate = 3600; // Revalidate every hour
export const dynamic = 'force-dynamic'; // Always dynamic

// Function-level caching
import { unstable_cache } from 'next/cache';
const getCachedData = unstable_cache(fetchData, ['cache-key'], { revalidate: 3600 });
```

## Route Segment Config

```typescript
export const dynamic = 'auto' | 'force-dynamic' | 'error' | 'force-static';
export const revalidate = false | 0 | number;
export const runtime = 'nodejs' | 'edge';
export const fetchCache = 'auto' | 'force-no-store' | 'force-cache';
```

## Middleware

```typescript
// middleware.ts (project root)
import { NextResponse, type NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Auth checks, redirects, headers
  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

## Metadata API

```typescript
// Static metadata
export const metadata: Metadata = {
  title: 'Page Title',
  description: 'Page description',
  openGraph: { title: '...', description: '...', images: ['...'] },
};

// Dynamic metadata
export async function generateMetadata({ params }): Promise<Metadata> {
  const data = await fetchData(params.id);
  return { title: data.title, description: data.description };
}
```

## next/image

```typescript
import Image from 'next/image';

// Fixed size
<Image src="/hero.png" width={800} height={400} alt="Hero" priority />

// Fill container
<div className="relative w-full h-64">
  <Image src="/bg.png" fill alt="Background" className="object-cover" />
</div>
```

Required: `alt` always. `width` + `height` OR `fill`. Use `priority` for LCP image.

## next/font

```typescript
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function Layout({ children }) {
  return <body className={inter.className}>{children}</body>;
}
```

## next/link

```typescript
import Link from 'next/link';

<Link href="/dashboard">Dashboard</Link>
<Link href="/blog/[slug]" as={`/blog/${post.slug}`}>Read more</Link>
```
