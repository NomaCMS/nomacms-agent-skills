---
name: noma-mcp-content-structure
description: >-
  Builds Noma-backed sites schema-first using the Noma/NomaCMS MCP — create
  collections, fields, and seed content before or while implementing pages. Use
  when the user builds a site, landing page, or CMS-driven UI with Noma (Next.js,
  Nuxt, Astro, or any @nomacms/js-sdk app) and has the Noma MCP enabled; prevents
  shipping only hardcoded sections with no CMS structure.
---

# Noma MCP: Content Structure First

When the user asks to build a **website, landing page, or app backed by Noma** and the **Noma (NomaCMS) MCP** is available, **CMS structure is part of the deliverable**. Implementing header, hero, features, and footer **only as static UI (React, Vue, Astro, plain HTML, etc.) with hardcoded copy** — without creating the matching **collections and fields** in Noma — is incomplete work.

## Mandatory workflow

1. **Infer schema from the page plan**  
   Map each section to collections: singletons for one-off page copy (hero, footer blurb, global nav labels) vs repeating rows (feature cards, testimonials, footer links). Choose **stable slugs** (kebab-case) the app will query. **Do not** add custom “published at” date fields for entries that only need a public timestamp — Noma already exposes **`published_at`** on every entry; duplicating it in **`fields`** is a common agent mistake (see **`noma-content`** and **`noma-collections-fields`**).

2. **Use the Noma MCP before finishing the UI**  
   - List and read each tool’s JSON schema under your MCP descriptors folder **before** calling tools (required parameter names differ per server).  
   - For **multilingual** apps, define project **locales** first (`add_project_locale` per code, optionally `set_default_project_locale`), or use the **REST** routes in **`noma-sdk-setup`** — the content API’s `locale` parameter only works for locales already on the project.  
   - Create **collections** (`is_singleton: true` where there is exactly one document per site or per page).  
   - Add **fields** for every piece of editable content (titles, subtitles, CTAs, URLs, richtext body, relations to assets if applicable).  
   - If the MCP exposes content/entry tools, **seed at least one published (or draft) entry** per collection so the app has real data to render.

3. **Then implement the frontend**  
   Wire the stack the user is using: follow **`noma-nextjs`**, **`noma-nuxt`**, or **`noma-astro`** when applicable; otherwise use **`noma-sdk-setup`** + **`noma-content`** with `@nomacms/js-sdk`. Fetch by the **exact collection slugs** you created; types or field access must match the schema you defined. For **`is_singleton: true`** collections, **`content.list(slug)`** returns **one object**, not a paginated list — see **`noma-content`** (list response matrix). Do not use **`paginate`** for singletons.

4. **If the Noma MCP is missing or fails**  
   Say so clearly. Fall back to **`noma-collections-fields`** with a server-only `apiKey` (one-off script, API route, Nitro/server handler, server action, etc.) — same schema plan, SDK instead of MCP.

## Checklist (copy and complete)

```text
Noma structure for this page/site:
- [ ] Collections planned (slug + singleton vs list)
- [ ] No redundant `published-at`-style fields (use `entry.published_at` unless a distinct editorial date is required)
- [ ] Fields defined per collection
- [ ] MCP tools invoked (or SDK fallback documented)
- [ ] Optional: seed entries for smoke-test
- [ ] App reads live slugs (SDK), not only placeholders
```

## References

- Section-to-schema patterns and examples: [reference.md](reference.md)
- Field types, `collections.*` / `fields.*` payloads: **`noma-collections-fields`**
- Listing and rendering entries: **`noma-content`**
- Framework guides: **`noma-nextjs`**, **`noma-nuxt`**, **`noma-astro`**
