# Nuxt Integration Reference

## Server Client Utility (Nitro)

```typescript
import { createClient } from '@nomacms/js-sdk';

let _client: ReturnType<typeof createClient> | null = null;

export function useNomaServer() {
  if (!_client) {
    const config = useRuntimeConfig();
    _client = createClient({
      projectId: config.nomaProjectId,
      apiKey: config.nomaApiKey,
    });
  }
  return _client;
}
```

## Runtime Config

```typescript
export default defineNuxtConfig({
  runtimeConfig: {
    nomaProjectId: process.env.NOMA_PROJECT_ID,
    // Sanctum PAT from User settings → API keys
    nomaApiKey: process.env.NOMA_API_KEY,
    public: {
      nomaProjectId: process.env.NOMA_PROJECT_ID,
    },
  },
});
```

## Content `list()` response shapes

**`is_singleton: true`** → `content.list(slug)` is **one entry object** (not `data[]`, not paginated). **`is_singleton: false`** + **`paginate`** → `{ data, meta, links }`. See **`noma-content`** for the full matrix before normalizing with `Array.isArray(res) ? res : res.data`.

## Server Routes

### Read list

```typescript
export default defineEventHandler(async () => {
  const client = useNomaServer();
  return client.content.list('blog-posts', {
    state: 'published',
    sort: 'created_at:desc',
    paginate: 10,
  });
});
```

### Read detail

```typescript
export default defineEventHandler(async (event) => {
  const uuid = getRouterParam(event, 'uuid')!;
  const client = useNomaServer();
  return client.content.get('blog-posts', uuid);
});
```

### Create

```typescript
export default defineEventHandler(async (event) => {
  const body = await readBody(event);
  const client = useNomaServer();
  return client.content.create('blog-posts', {
    data: body,
    state: 'draft',
  });
});
```

## SSG / ISR

### Generate static output

```bash
npx nuxi generate
```

### Route rules

```typescript
export default defineNuxtConfig({
  routeRules: {
    '/posts/**': { isr: 60 },
    '/about': { prerender: true },
  },
});
```

### Prerender dynamic routes

```typescript
export default defineNuxtConfig({
  hooks: {
    async 'prerender:routes'(ctx) {
      const res = await createClient({ /* ... */ }).content.list('blog-posts', {
        state: 'published',
        paginate: 100,
        page: 1,
      });
      const entries = Array.isArray(res) ? res : (res as { data: Array<{ uuid: string }> }).data;
      for (const post of entries) ctx.routes.add(`/posts/${post.uuid}`);
    },
  },
});
```

## Social login (Nitro)

After the browser completes Google (or another supported) OAuth, **POST** the provider `id_token` to a Nitro API route. Use `createClient({ projectId })` **without** `apiKey` and call `signInWithSocial({ provider: 'google', id_token, nonce? })`, then set an HTTP-only session cookie. See `noma-user-auth` and `noma-nextjs` examples for the same pattern.

## Auth Guard + Pinia

Use route middleware for token checks and a Pinia store that wraps `signInWithPassword`, `signInWithSocial`, `me`, and `signOut` for app-level auth state.
