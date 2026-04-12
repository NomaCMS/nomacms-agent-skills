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
