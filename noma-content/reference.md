# Content API Reference

## `list(collectionSlug, params?)`

```typescript
client.content.list(collectionSlug: string, params?: {
  state?: 'draft' | 'published';
  locale?: string;
  exclude?: string | string[];
  where?: Record<string, unknown>;
  sort?: string;                 // 'field:asc' or 'field:desc'
  limit?: number;
  offset?: number;
  paginate?: number;
  page?: number;
  timestamps?: boolean;
  count?: boolean;
  first?: boolean;
})
```

### Singleton vs list collections (`GET /api/{collection}`)

The server branches on **`is_singleton`** from the collection record:

1. **Singleton (`is_singleton: true`)**  
   - Returns **one** `ContentEntry` JSON object at the **root** (same shape as `get()`): `uuid`, `locale`, `published_at`, `fields`.  
   - **Does not** return an array, pagination metadata, or `{ count }`. Query params **`paginate`**, **`count`**, **`limit`**, **`offset`**, and **`first` are not used** for this branch.  
   - Without `locale`, the query is scoped to the **project default locale**.  
   - **404** if no entry exists for the resolved locale/state filters.

2. **Non-singleton (`is_singleton: false`)**  
   - **`count`** (present in query): response is **`{ count: number }` only**.  
   - **`paginate`**: Laravel-style page — **`{ data: Entry[], links, meta }`** (entries in **`data`**). Use **`page`** for the page index.  
   - **Neither `paginate` nor `count`**: default is **all rows** as a **JSON array** `[ Entry, … ]` at the **root** (not wrapped in `data`).  
   - **`first: true`**: **one** `Entry` object at the **root** (first row after filters/sort).  
   - **`limit` / `offset`**: apply only when **`paginate` is not set**; **`offset` requires `limit`**.

**Agent rule:** read **`is_singleton`** from `collections.get(slug)` before interpreting `list()` results. Treating a singleton like a paginated list (e.g. expecting `response.data[0]` or `meta`) is wrong.

**Singleton translations:** one entry per locale; **`POST /api/{collection}`** (and bulk create) **auto-links** new singleton rows to existing entries in other locales (shared **`translation_group_id`**). Use **`GET /api/{collection}/{uuid}?translation_locale=xx`** to fetch the linked row for another locale.

### Pagination

- Offset-based: `limit` + `offset` (non-singleton, no `paginate`)
- Page-based: `paginate` + `page` (non-singleton)

### Sorting

- Ascending: `price:asc`
- Descending: `created_at:desc`

## `get(collectionSlug, uuid, params?)`

```typescript
client.content.get(collectionSlug: string, uuid: string, params?: {
  locale?: string;
  translation_locale?: string;
  state?: 'draft' | 'published';
  exclude?: string | string[];
  timestamps?: boolean;
})
```

## `linkTranslation(collectionSlug, uuid, payload)`

```typescript
client.content.linkTranslation(collectionSlug: string, uuid: string, payload: {
  translation_entry_uuid: string;
})
```

**HTTP:** `POST /api/{collection}/{uuid}/link-translation` with JSON body `{ "translation_entry_uuid": "<uuid>" }`.

**Purpose:** Put two existing entries in the same collection into one **translation group** so they differ by **locale** and `get(…, { translation_locale })` can load the linked row. **422** if the target is missing, same locale as the anchor, or the anchor itself.

**Auth:** API key must include the **`update`** ability (same as entry updates).

**Response (success):** `{ message: string, translation_group_id: number }`.

Singleton creates already auto-link; use this mainly for **repeating** collections when you created locale rows separately.

## Create / Update / Patch Payload

```typescript
{
  data: Record<string, unknown>;
  locale?: string;
  state?: 'draft' | 'published';
  published_at?: string; // ISO 8601
}
```

## Bulk Methods

- `bulkCreate(collectionSlug, { items })`
- `bulkUpdate(collectionSlug, { items })`
- `bulkDelete(collectionSlug, { uuids, force? })`

## `where` Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `{ title: { eq: 'Hello' } }` |
| `not` | Not equals | `{ title: { not: 'Hello' } }` |
| `like` | Contains (uses `%`) | `{ title: { like: 'shirt' } }` |
| `in` | In list | `{ status: { in: 'active,pending' } }` |
| `not_in` | Not in list | `{ status: { not_in: 'archived' } }` |
| `gt` | Greater than | `{ price: { gt: 50 } }` |
| `gte` | Greater than or equal | `{ price: { gte: 50 } }` |
| `lt` | Less than | `{ price: { lt: 100 } }` |
| `lte` | Less than or equal | `{ price: { lte: 100 } }` |
| `between` | Between two values | `{ price: { between: '30,120' } }` |
| `not_between` | Outside range | `{ price: { not_between: '30,120' } }` |
| `null` | Is null | `{ published_at: { null: true } }` |
| `not_null` | Is not null | `{ published_at: { not_null: true } }` |

## OR and Relation Filters

- OR groups:
  - `where: { or: [{ category: { eq: 'shoes' } }, { category: { eq: 'boots' } }] }`
- Relation filters:
  - `where: { category: { name: 'Apparel' } }`

## Filterable Core Columns

`id`, `uuid`, `locale`, `state`, `created_at`, `updated_at`, `published_at`, plus custom collection fields.

## Entry JSON shape

Each entry has **`fields`**: `{ [fieldName]: value }` for custom fields. **`data`** is only the key in **request bodies** for create/update/patch (`{ data: { title: '...' } }`), not a property on returned entries.

## Richtext fields

- **Write:** always a **single markdown string** in `data` (including inside repeatable groups).
- **Read:** if `editor.outputFormat` is **`markdown`**, the API returns the markdown string; if **`html`** (default), it returns **HTML rendered** from that markdown (sanitized). Inspect the field definition via `collections.get(slug)` to know which you will receive.
