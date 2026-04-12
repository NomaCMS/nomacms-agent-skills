# Content API Examples

## List Published Posts

```typescript
const posts = await client.content.list('blog-posts', {
  state: 'published',
  sort: 'created_at:desc',
  paginate: 10,
  page: 1,
});
// Non-singleton + paginate → { data: [...], meta, links }
```

## Singleton collection (settings, homepage, global nav)

Use **`collections.get('settings')`** and confirm **`is_singleton: true`**, then list **without** `paginate`:

```typescript
const settings = await client.content.list('settings', { state: 'published' });
// One object: settings.fields.site_name — not settings.data[0], not an array
```

With an explicit locale:

```typescript
const settingsFr = await client.content.list('settings', {
  state: 'published',
  locale: 'fr',
});
```

## Count only (non-singleton)

```typescript
const { count } = await client.content.list('products', {
  state: 'published',
  where: { in_stock: { eq: true } },
  count: true,
});
// { count: number } — not an array of entries
```

## Get a Single Entry

```typescript
const post = await client.content.get('blog-posts', 'entry-uuid', {
  state: 'published',
  locale: 'en',
});
// post.fields.title — custom fields live under `fields`, not `data`
```

## Create, Update, Patch, Delete

```typescript
await client.content.create('blog-posts', {
  data: { title: 'My Post', body: '## Intro\n\nMarkdown body for richtext fields.' },
  state: 'draft',
  locale: 'en',
});

await client.content.update('blog-posts', 'entry-uuid', {
  data: { title: 'Updated Title', body: 'New body', category: 'tech' },
  state: 'published',
});

await client.content.patch('blog-posts', 'entry-uuid', {
  data: { title: 'Just the title changed' },
});

await client.content.delete('blog-posts', 'entry-uuid', true);
```

## Link two entries as translations (repeating collection)

After creating separate rows per locale, merge them so `get` with `translation_locale` works:

```typescript
await client.content.linkTranslation('blog-posts', englishUuid, {
  translation_entry_uuid: frenchUuid,
});
// { message, translation_group_id } — API key needs `update` ability
```

## Filtering

```typescript
await client.content.list('products', {
  where: {
    category: { eq: 'shoes' },
    price: { gte: 20, lte: 100 },
    published_at: { not_null: true },
  },
});
```

## OR Group

```typescript
await client.content.list('products', {
  where: {
    or: [{ category: { eq: 'shoes' } }, { category: { eq: 'boots' } }],
  },
});
```

## Relation Filter

```typescript
await client.content.list('products', {
  where: {
    category: { name: 'Apparel' },
  },
});
```

## Bulk Operations

```typescript
await client.content.bulkCreate('blog-posts', {
  items: [
    { data: { title: 'Post 1' }, state: 'draft' },
    { data: { title: 'Post 2' }, state: 'published', locale: 'en' },
  ],
});

await client.content.bulkUpdate('blog-posts', {
  items: [
    { uuid: 'uuid-1', data: { title: 'Updated 1' }, state: 'published' },
    { uuid: 'uuid-2', data: { title: 'Updated 2' } },
  ],
});

await client.content.bulkDelete('blog-posts', {
  uuids: ['uuid-1', 'uuid-2', 'uuid-3'],
  force: false,
});
```
