---
name: noma-agent-skills
description: >-
  Routes Noma CMS (@nomacms/js-sdk) work to the correct skill package in this
  repository. Use when the user works with Noma, nomacms, this skills repo, or
  asks about SDK setup, content, assets, auth, collections, MCP content
  structure, errors, Next.js, Nuxt, or Astro integration.
---

## Skill packages (workspace root)

| Directory | Use when |
|-----------|----------|
| `noma-sdk-setup/` | Installing SDK, client, tokens, env, connectivity, **project locales API** |
| `noma-content/` | Content CRUD, filtering, pagination, bulk ops — **singleton vs list response shapes** |
| `noma-assets/` | Uploads, asset metadata, bulk asset ops |
| `noma-user-auth/` | Sign up/in, refresh, auth state, user API keys |
| `noma-auth-contract/` | Canonical frontend auth contract and must-pass integration checklist |
| `noma-nextjs-auth/` | Next.js/NextAuth-specific auth implementation guardrails |
| `noma-auth-review/` | Audit existing auth implementations against Noma auth contract |
| `noma-collections-fields/` | Collections and field schema |
| `noma-mcp-content-structure/` | Schema-first sites with Noma MCP (any framework) |
| `noma-errors/` | Error types and try/catch patterns |
| `noma-nextjs/` | Next.js (RSC, SSG/ISR, server actions) |
| `noma-nuxt/` | Nuxt (server routes, Pinia, Nitro) |
| `noma-astro/` | Astro (static, SSR, content layer) |

## Workflow

1. Pick the table row that matches the task (or ask if unclear).
2. Read that package’s `SKILL.md` (for example `noma-content/SKILL.md`) first.
3. Follow links from that file to `reference.md` or `examples.md` only when extra detail is required.

## Auth routing order (important)

When the task is about project end-user auth integration quality (especially logout/refresh/session behavior):

1. Read `noma-auth-contract/SKILL.md` first.
2. Then read framework-specific auth skill (for example `noma-nextjs-auth/SKILL.md`).
3. Use `noma-auth-review/SKILL.md` for audits of existing implementations.
