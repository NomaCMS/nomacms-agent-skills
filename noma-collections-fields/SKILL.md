---
name: noma-collections-fields
description: >-
  Manages Noma collections and fields — create, update, delete, and reorder
  collections and their field schemas. Use when programmatically managing content
  schemas, field types, or collection settings.
---

# Noma Collections & Fields API

Collections and fields are managed via `client.collections.*` and `client.fields.*`. All operations require an API key (`apiKey`).

## Quick Start

```typescript
await client.collections.create({
  name: 'Blog Posts',
  slug: 'blog-posts',
  description: 'Articles and blog entries',
  is_singleton: false,
});
await client.collections.update('blog-posts', {
  name: 'Articles',
  description: 'Updated description',
});
await client.collections.delete('blog-posts');
await client.fields.create('blog-posts', {
  name: 'Title',
  slug: 'title',
  type: 'text',
  required: true,
  options: {
    placeholder: 'Enter post title',
  },
});
await client.fields.update('blog-posts', fieldId, {
  name: 'Post Title',
  required: true,
  options: {
    placeholder: 'Updated placeholder',
  },
});
await client.fields.delete('blog-posts', fieldId);
await client.fields.reorder('blog-posts', {
  fields: [
    { uuid: 'field-uuid-a', order: 0 },
    { uuid: 'field-uuid-b', order: 1 },
  ],
});
```

Use `collections.get()` to inspect current fields before schema updates.

**`is_singleton` drives list API behavior:** when **`is_singleton` is `true`**, `client.content.list(slug)` returns **one entry object** at the root (not an array, not paginated `{ data, meta }`). **`paginate` does not apply** to singletons. When **`is_singleton` is `false`**, `list()` can return a **top-level array**, a **paginated** payload with **`data` + `meta` + `links`**, **`{ count }`**, or a **single** entry with **`first: true`** — see **`noma-content`** for the matrix. Always check **`is_singleton`** before writing fetch/parsing code.

Richtext fields use **`options.editor.outputFormat`**: `'html'` (default) or `'markdown'`. That controls how the Content API **returns** the field; **input** is always a markdown string. See [reference.md](reference.md).

## References

- Full collections/fields method matrix: [reference.md](reference.md)
- Copy-ready schema management snippets: [examples.md](examples.md)
- Error handling patterns: `noma-errors`
