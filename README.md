# Noma Agent Skills

Agent skills for [`@nomacms/js-sdk`](https://www.npmjs.com/package/@nomacms/js-sdk) and [Noma CMS](https://nomacms.com), organized for the [`skills`](https://www.npmjs.com/package/skills) ecosystem.

This repository uses root-level skill directories so any supported agent can install from the same source.

## Skills Included

| Skill | Description |
|-------|-------------|
| **noma-sdk-setup** | Client creation, tokens, env vars, TokenStorageAdapter, **project locales (REST + admin key)** |
| **noma-content** | Content CRUD, filtering, pagination, bulk operations — singleton vs list **list()** response shapes |
| **noma-assets** | Asset upload, metadata, bulk operations |
| **noma-user-auth** | Sign up/in (password + social `id_token`), refresh, auth state, user API keys |
| **noma-auth-contract** | Canonical frontend auth contract and must-pass integration checklist |
| **noma-nextjs-auth** | Next.js / NextAuth-specific auth implementation guardrails |
| **noma-auth-review** | Audit existing auth implementations against the Noma auth contract |
| **noma-collections-fields** | Collection and field schema management |
| **noma-mcp-content-structure** | Schema-first Noma sites via MCP — collections/fields before UI (Next.js, Nuxt, Astro, or other SDK apps) |
| **noma-errors** | Error hierarchy and try/catch patterns |
| **noma-nextjs** | Next.js integration (RSC, SSG/ISR, server actions) |
| **noma-nuxt** | Nuxt integration (server routes, Pinia, Nitro) |
| **noma-astro** | Astro integration (static, SSR, content layer) |

## Quick Start

```bash
# List skills available in this repository
npx skills add nomacms/noma-agent-skills --list

# Install all skills for detected agents
npx skills add nomacms/noma-agent-skills --skill '*'

# Install specific skills
npx skills add nomacms/noma-agent-skills --skill noma-sdk-setup --skill noma-content

# Install only for Cursor
npx skills add nomacms/noma-agent-skills --skill '*' -a cursor
```

Project-local install is the default. Use `-g` for global installs.

## Verify Discovery

From this repository root:

```bash
npx skills add . --list
```

If discovery works, you should see all Noma skills listed.

## Repository Structure

```text
noma-agent-skills/
├── README.md
├── SKILL.md                    # router: which skill package to open
├── noma-sdk-setup/
│   ├── SKILL.md
│   ├── reference.md
│   └── examples.md
├── noma-content/
├── noma-assets/
├── noma-user-auth/
├── noma-auth-contract/
├── noma-nextjs-auth/
├── noma-auth-review/
├── noma-collections-fields/
├── noma-mcp-content-structure/
├── noma-errors/
├── noma-nextjs/
├── noma-nuxt/
└── noma-astro/
```

Each skill directory must contain `SKILL.md` with YAML frontmatter (`name`, `description`).
The repository root may include `SKILL.md` as a router to skill packages (see that file).

## Authoring Guidelines

Edit files directly under `<skill-name>/`.

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

- **Every project**: `noma-sdk-setup`, `noma-errors`
- **Content-driven sites**: add `noma-content`
- **Media-heavy apps**: add `noma-assets`
- **User-facing apps**: add `noma-user-auth`; for contract checks and audits add `noma-auth-contract`, framework auth skills (e.g. `noma-nextjs-auth`), and `noma-auth-review` when relevant
- **Schema management**: add `noma-collections-fields`
- **Cursor + Noma MCP**: add `noma-mcp-content-structure` so pages are not built as static-only sections (any framework)
- **Framework-specific**: add one of `noma-nextjs`, `noma-nuxt`, or `noma-astro`
