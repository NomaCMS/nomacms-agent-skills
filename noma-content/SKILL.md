---
name: noma-content
description: >-
  Works with Noma content entries — list, get, create, update, patch,
  delete, bulk operations, and optional translation linking. Use when building pages that display, filter,
  paginate, or mutate content from Noma collections.
---

# Noma Content API

All methods are on `client.content.*`. Requires an API key (`apiKey` from **User settings → API keys**).

## Response shape

List and get responses expose each entry as **`uuid`**, **`locale`**, **`published_at`**, and **`fields`** (an object of custom field values). There is no per-entry `data` property for field values — use **`entry.fields.title`**, not `entry.data.title`.

**Richtext fields** are stored as **markdown**. On **create/update/patch**, send a **string** (markdown). On **read**, the value is either **markdown** or **HTML** depending on the field’s `options.editor.outputFormat` in the collection schema (`markdown` vs `html`, default `html`).

## List response by collection type (critical)

**Always** check the collection’s **`is_singleton`** flag (via `client.collections.get(slug)` or your schema). The **same** `client.content.list(slug, …)` call returns **different JSON shapes**:

| Collection | HTTP / JSON shape | Notes |
|------------|---------------------|--------|
| **`is_singleton: true`** | **One object** at the root: `{ uuid, locale, published_at, fields }` | **Not** an array. **Not** Laravel pagination (`data` + `meta` + `links`). **`paginate`, `count`, `limit`/`offset`, and `first` do not apply** — the server handles singletons in a separate branch and returns the single entry (or **404**). |
| **`is_singleton: false`** + **`paginate`** | **`{ data: Entry[], links, meta }`** | Page with `page` query param. Entries are in **`data`**. |
| **`is_singleton: false`** (no `paginate`) | **Top-level JSON array** `[ Entry, … ]` | All matching rows (subject to `limit`/`offset` when both are set). |
| **`is_singleton: false`** + **`first: true`** | **One object** at the root (same as `get`) | First matching row only. |
| **`is_singleton: false`** + **`count`** (any value) | **`{ count: number }`** | Count only — **no entries**. Evaluated only for non-singleton collections. |

**Singleton locale:** if you omit `locale` on a singleton list, the API uses the **project default locale**. Pass `locale` explicitly to read another language’s singleton row.

**Do not** pass `paginate` for singleton collections expecting pagination metadata — it is **ignored** server-side; you still receive a **single entry object**.

### Singleton + multiple project locales

For **`is_singleton: true`**, the CMS allows **at most one entry per locale**. When you **create** entries via the API (including **bulk create**), new rows are **automatically linked** into one **translation group** so `GET …/{uuid}?translation_locale=…` resolves sibling locales. You do not need a separate “link translation” call for normal creates. Duplicate **`POST`** for the same locale still returns **422**.

**Repeating collections (`is_singleton: false`):** entries are **not** auto-linked across locales. After you have two rows (e.g. English and French posts), call **`client.content.linkTranslation(slug, anchorUuid, { translation_entry_uuid: otherUuid })`** — `POST /api/{collection}/{uuid}/link-translation` — so `translation_locale` on **`get`** resolves the sibling. Requires an API key with **`update`** ability. The API only **links**; it does not expose unlink (use the dashboard).

## Quick Start

```typescript
const posts = await client.content.list('blog-posts', {
  state: 'published',
  sort: 'created_at:desc',
  paginate: 10,
});
```

Singleton example (e.g. site settings slug `settings`):

```typescript
const settings = await client.content.list('settings', { state: 'published' });
// `settings` is one entry object: settings.fields.site_name, not settings[0], not settings.data
```

## Core Operations

```typescript
const item = await client.content.get('blog-posts', 'entry-uuid');

await client.content.create('blog-posts', {
  data: { title: 'My Post', body: '## Markdown body\n\nParagraph.' },
  state: 'draft',
});

await client.content.patch('blog-posts', 'entry-uuid', {
  data: { title: 'Updated title' },
});

await client.content.delete('blog-posts', 'entry-uuid');
```

## Filtering

Use a `where` object on `list()`:

```typescript
await client.content.list('products', {
  where: {
    category: { eq: 'shoes' },
    price: { gte: 20, lte: 100 },
  },
});
```

Supported operators include `eq`, `not`, `like`, `in`, `gt`, `gte`, `lt`, `lte`, `between`, `null`, and `not_null`.

## Bulk Operations

Use bulk methods for batches:

```typescript
await client.content.bulkCreate('blog-posts', { items: [{ data: { title: 'Post 1' } }] });
await client.content.bulkUpdate('blog-posts', { items: [{ uuid: 'uuid-1', data: { title: 'Updated' } }] });
await client.content.bulkDelete('blog-posts', { uuids: ['uuid-1', 'uuid-2'] });
```

## Project locales (i18n)

Content requests use a **`locale`** query/body field only for locale codes that exist on the project. **Configure locales on the project** (dashboard **Localization**, or **`POST /api/project/locales`** with an **`admin`** API key — see **`noma-sdk-setup`**) before building a multi-language Next.js or other app.

## References

- Full API signatures and parameter tables: [reference.md](reference.md)
- Copy-ready query and mutation snippets: [examples.md](examples.md)
- Error handling patterns: `noma-errors`
