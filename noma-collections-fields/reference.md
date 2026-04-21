# Collections & Fields Reference

## Collections Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `list` | `() => Promise` | List all collections |
| `get` | `(slug: string) => Promise` | Get collection with fields |
| `create` | `(payload: Record<string, unknown>) => Promise` | Create collection |
| `update` | `(slug: string, payload: Record<string, unknown>) => Promise` | Update collection |
| `delete` | `(slug: string) => Promise` | Delete collection |
| `reorder` | `(payload: { collections: { uuid: string; order: number }[] }) => Promise` | Reorder collections (`uuid` + `order`, min 0) |

## Fields Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `create` | `(slug: string, payload: Record<string, unknown>) => Promise` | Add field to collection |
| `update` | `(slug: string, fieldId: number \| string, payload: Record<string, unknown>) => Promise` | Update field |
| `delete` | `(slug: string, fieldId: number \| string) => Promise` | Delete field |
| `reorder` | `(slug: string, payload: { fields: { uuid: string; order: number }[] }) => Promise` | Reorder fields (`uuid` + `order`, min 0) |

## Notes

- There is no standalone `fields.list()`.
- Use `collections.get(slug)` to inspect fields.
- `fieldId` accepts `number | string` where applicable.

## Singleton Collections

Use `is_singleton: true` for one-entry configuration content (for example site settings, single homepage document per locale).

**Content API:** `GET /api/{slug}` for a singleton returns **one JSON object** (the entry), not a list and not pagination. **`paginate` and `count` query params do not apply.** Omitting `locale` uses the project default locale. See skill **`noma-content`** (“List response by collection type”).

## Richtext field options

Richtext fields use markdown storage. When creating or updating a field of type `richtext`, set:

```typescript
options: {
  editor: {
    type: 1,
    outputFormat: 'html' | 'markdown', // default API/read behavior for this field
  },
}
```

- **`markdown`** — Content API returns the **raw markdown** string for this field.
- **`html`** — API returns **HTML** generated from that markdown (sanitized).

On **writes** (create/update content), always send richtext values as a **string** (markdown), never Lexical JSON or `{ html, json }` objects.

## Relation fields (`type: 'relation'`)

Relation fields point to **other content entries** in the same project. Define them with `options.relation`:

```typescript
await client.fields.create('products', {
  name: 'Category',
  slug: 'category',
  type: 'relation',
  required: false,
  options: {
    relation: {
      /** Target collection: positive integer id, or slug / name string (normalized server-side to an id) */
      collection: 'categories',
      /** 1 = one-to-one (single related entry), 2 = one-to-many (ordered list) */
      type: 1,
    },
    /** Optional: when true, relation pickers / queries may include draft entries (dashboard behavior). */
    includeDraft: false,
  },
});
```

- After create/update, the API stores `options.relation.collection` as a **numeric collection id** when the target resolves inside the project (slug/name strings are accepted on input).
- **Cardinality:** `type: 1` — reads return **one** nested entry object (or `null`). `type: 2` — reads return an **array** of nested entry objects.
- Content **writes** must send **entry UUID strings** and/or **numeric entry ids**, not full entry objects from `GET` responses. See **`noma-content`** (“Relation fields” on reads vs writes).
