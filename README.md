# NomaCMS Agent Skills

Agent skills for [`@nomacms/js-sdk`](https://www.npmjs.com/package/@nomacms/js-sdk) and [NomaCMS](https://nomacms.com), organized for the [`skills`](https://www.npmjs.com/package/skills) ecosystem.

This repository follows the [`skills` CLI](https://www.npmjs.com/package/skills) layout: one directory per skill under `skills/`, so any supported agent can install from the same source.

## Skills Included

| Skill | Description |
|-------|-------------|
| **nomacms-sdk-setup** | Client creation, tokens, env vars, TokenStorageAdapter, **project locales (REST + admin key)** |
| **nomacms-content** | Content CRUD, filtering, pagination, bulk operations — singleton vs list **list()** response shapes |
| **nomacms-assets** | Asset upload, metadata, bulk operations |
| **nomacms-user-auth** | Sign up/in (password + social `id_token`), refresh, auth state, user API keys |
| **nomacms-auth-contract** | Canonical frontend auth contract and must-pass integration checklist |
| **nomacms-nextjs-auth** | Next.js / NextAuth-specific auth implementation guardrails |
| **nomacms-auth-review** | Audit existing auth implementations against the NomaCMS auth contract |
| **nomacms-collections-fields** | Collection and field schema management |
| **nomacms-mcp-content-structure** | Schema-first NomaCMS sites via MCP — collections/fields before UI (Next.js, Nuxt, Astro, or other SDK apps) |
| **nomacms-errors** | Error hierarchy and try/catch patterns |
| **nomacms-nextjs** | Next.js integration (RSC, SSG/ISR, server actions) |
| **nomacms-nuxt** | Nuxt integration (server routes, Pinia, Nitro) |
| **nomacms-astro** | Astro integration (static, SSR, content layer) |

## Quick Start

```bash
# List skills available in this repository
npx skills add nomacms/nomacms-agent-skills --list

# Install all skills for detected agents
npx skills add nomacms/nomacms-agent-skills --skill '*'

# Install specific skills
npx skills add nomacms/nomacms-agent-skills --skill nomacms-sdk-setup --skill nomacms-content

# Install only for Cursor
npx skills add nomacms/nomacms-agent-skills --skill '*' -a cursor
```

Project-local install is the default. Use `-g` for global installs.

## Verify Discovery

From this repository root:

```bash
npx skills add . --list
```

If discovery works, you should see all NomaCMS skills listed.

## Repository Structure

```text
nomacms-agent-skills/
├── README.md
└── skills/
    ├── nomacms-sdk-setup/
    │   ├── SKILL.md
    │   ├── reference.md
    │   └── examples.md
    ├── nomacms-content/
    ├── nomacms-assets/
    ├── nomacms-user-auth/
    ├── nomacms-auth-contract/
    ├── nomacms-nextjs-auth/
    ├── nomacms-auth-review/
    ├── nomacms-collections-fields/
    ├── nomacms-mcp-content-structure/
    ├── nomacms-errors/
    ├── nomacms-nextjs/
    ├── nomacms-nuxt/
    └── nomacms-astro/
```

Each skill directory must contain `SKILL.md` with YAML frontmatter (`name`, `description`).
The CLI discovers skills under the `skills/` directory (not at the repository root).

## Authoring Guidelines

Edit files directly under `skills/<skill-name>/`.

Recommended pattern per skill:
- `SKILL.md` (required)
- `reference.md` (optional)
- `examples.md` (optional)

When writing `SKILL.md`:
- Keep `name` stable and lowercase with hyphens.
- Make `description` explicit about when the skill should be used.
- Keep examples practical and SDK-accurate.

## Picking Skills

Install only the skills relevant to your project:

- **Every project**: `nomacms-sdk-setup`, `nomacms-errors`
- **Content-driven sites**: add `nomacms-content`
- **Media-heavy apps**: add `nomacms-assets`
- **User-facing apps**: add `nomacms-user-auth`; for contract checks and audits add `nomacms-auth-contract`, framework auth skills (e.g. `nomacms-nextjs-auth`), and `nomacms-auth-review` when relevant
- **Schema management**: add `nomacms-collections-fields`
- **Cursor + NomaCMS MCP**: add `nomacms-mcp-content-structure` so pages are not built as static-only sections (any framework)
- **Framework-specific**: add one of `nomacms-nextjs`, `nomacms-nuxt`, or `nomacms-astro`
