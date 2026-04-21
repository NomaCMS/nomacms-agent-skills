---
name: noma-sdk-setup
description: >-
  Sets up and configures the Noma JS SDK client. Use when installing
  @nomacms/js-sdk, creating a client, configuring API keys, environment
  variables, or troubleshooting SDK connectivity.
---

# Noma SDK Setup

## Installation

```bash
npm install @nomacms/js-sdk
```

## Quick Start

```typescript
import { createClient } from '@nomacms/js-sdk';

const client = createClient({
  projectId: 'your-project-uuid',
  apiKey: 'your-api-key',
  timeout: 30000, // optional, defaults to 30000ms
});
```

## Authentication Modes

### API key (server/admin)

`apiKey` is an API key from the Noma dashboard (**User settings → API keys**; Sanctum personal access token). Use with `projectId` (Project UUID, often copied from **Project settings → API Access**). Prefer env **`NOMA_API_KEY`** (same name as the Noma MCP server).

```typescript
const client = createClient({
  projectId: process.env.NOMA_PROJECT_ID!,
  apiKey: process.env.NOMA_API_KEY!,
});
```

### Project user auth (browser/end-user)

```typescript
const client = createClient({
  projectId: process.env.NEXT_PUBLIC_NOMA_PROJECT_ID!,
  projectUserAuth: {
    autoRefresh: true,
    tokenStorage: myTokenStorage,
  },
});
```

## Project locales (multilingual apps)

Configure **which locale codes exist** on the project **before** creating translated entries. **Read** `locales` / `default_locale` via **`GET /api/`**. **Add or change** them with **`POST /api/project/locales`**, **`DELETE /api/project/locales/{locale}`**, **`PUT /api/project/locales/default`** (requires **`admin`** on the API key). Details and table: [reference.md](reference.md). Noma MCP exposes **`add_project_locale`** and **`set_default_project_locale`** only (locale removal is dashboard or direct REST — not MCP).

## Security Rules

- Never expose `NOMA_API_KEY` to browser code.
- Use `apiKey` for server-side content/schema/admin API calls.
- Use `projectUserAuth` for end-user sessions (password and/or social `id_token` via `signInWithSocial`).
- Prefer exchanging provider `id_token` for Noma session tokens on the **server** (Route Handler / server route), not from public client bundles.

## Debug Check

```typescript
console.log(client.getDebugInfo());
// {
//   basePath: 'https://app.nomacms.com/api',
//   projectId: 'xxxxxxxx-...',
//   timeout: 30000,
//   hasApiKey: true,
//   hasProjectUserToken: false,
// }
```

## References

- Full options/types/API surface: [reference.md](reference.md)
- Copy-ready adapters and setup snippets: [examples.md](examples.md)
- User auth flow details: `noma-user-auth`
- Error handling: `noma-errors`
