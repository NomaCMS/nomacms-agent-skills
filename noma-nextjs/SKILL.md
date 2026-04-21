---
name: noma-nextjs
description: >-
  Integrates Noma with Next.js — server components, SSG/ISR, server actions,
  route handlers, auth (password + social id_token), and middleware. Use when building a Next.js app that
  fetches or mutates content from Noma.
---

# Noma + Next.js

App Router is the primary focus.

## Quick Start

Use separate clients for server and browser contexts:

```typescript
// lib/noma-server.ts — server-side only, never imported in client components
import { createClient } from '@nomacms/js-sdk';

export const nomaServer = createClient({
  projectId: process.env.NOMA_PROJECT_ID!,
  // API key from User settings → API keys (not an end-user uak_ key)
  apiKey: process.env.NOMA_API_KEY!,
});
```

```typescript
// lib/noma-client.ts — browser-safe, no apiKey
import { createClient } from '@nomacms/js-sdk';

export function createBrowserClient() {
  return createClient({
    projectId: process.env.NEXT_PUBLIC_NOMA_PROJECT_ID!,
    projectUserAuth: {
      autoRefresh: true,
      tokenStorage: localStorageAdapter, // see noma-sdk-setup
    },
  });
}
```

Fetch in an async server component:

```tsx
// app/posts/page.tsx
import { nomaServer } from '@/lib/noma-server';

export default async function PostsPage() {
  const posts = await nomaServer.content.list('blog-posts', {
    state: 'published',
    sort: 'created_at:desc',
    paginate: 10,
  });

  return (
    <ul>
      {posts.data.map((post: { uuid: string; fields: { title?: string } }) => (
        <li key={post.uuid}>{post.fields.title}</li>
      ))}
    </ul>
  );
}
```

## Key Patterns

- Server components: direct `await` calls, no hooks
- Keep `NOMA_API_KEY` server-only (no `NEXT_PUBLIC_` prefix)
- Use `notFound()` from `next/navigation` when catching `NotFoundError` in dynamic pages
- Prefer route handlers or server actions for mutations

## References

- Full Next.js integration reference: [reference.md](reference.md)
- Copy-ready snippets: [examples.md](examples.md)
- Noma MCP + schema-first pages (any framework): `noma-mcp-content-structure`
- SDK setup and auth context: `noma-sdk-setup` and `noma-user-auth`
- Filtering and operators: `noma-content`
- Error patterns: `noma-errors`
