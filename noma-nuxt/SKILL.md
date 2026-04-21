---
name: noma-nuxt
description: >-
  Integrates Noma with Nuxt — server routes, composables, SSG/ISR, plugins,
  middleware, and Pinia stores. Use when building a Nuxt app that fetches or
  mutates content from Noma.
---

# Noma + Nuxt

## Quick Start

```typescript
// plugins/noma.client.ts
import { createClient, type TokenStorageAdapter } from '@nomacms/js-sdk';

export default defineNuxtPlugin(() => {
  const config = useRuntimeConfig();

  const tokenStorage: TokenStorageAdapter = {
    getAccessToken: () => localStorage.getItem('noma_access_token') ?? undefined,
    setAccessToken: (t) => t ? localStorage.setItem('noma_access_token', t) : localStorage.removeItem('noma_access_token'),
    getRefreshToken: () => localStorage.getItem('noma_refresh_token') ?? undefined,
    setRefreshToken: (t) => t ? localStorage.setItem('noma_refresh_token', t) : localStorage.removeItem('noma_refresh_token'),
    clear: () => { localStorage.removeItem('noma_access_token'); localStorage.removeItem('noma_refresh_token'); },
  };

  const client = createClient({
    projectId: config.public.nomaProjectId,
    projectUserAuth: { autoRefresh: true, tokenStorage },
  });

  return { provide: { noma: client } };
});
```

Access in components via `useNuxtApp().$noma`.

Proxy content through Nitro routes:

```typescript
// server/api/posts/index.get.ts
export default defineEventHandler(async () => {
  const client = useNomaServer();
  return client.content.list('blog-posts', {
    state: 'published',
    sort: 'created_at:desc',
    paginate: 10,
  });
});
```

## Key Patterns

- Keep `NOMA_API_KEY` (API key from **User settings → API keys**) in server-only `runtimeConfig` (not `public`)
- Use `server/utils/` for shared server-side client
- Proxy all content through `server/api/` routes to avoid exposing the API key
- Use route rules (`isr`, `prerender`) for cache and generation strategy

## References

- Full Nuxt integration reference: [reference.md](reference.md)
- Copy-ready snippets: [examples.md](examples.md)
- Noma MCP + schema-first pages: `noma-mcp-content-structure`
- Filters and operators: `noma-content`
- Error handling: `noma-errors`
