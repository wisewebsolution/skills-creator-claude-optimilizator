# Norsktorget API Pagination Guide

All endpoints that return a collection of resources **must** implement cursor-based pagination.

## Request

Clients should provide `limit` and `cursor` query parameters.

- `limit`: The number of items to return (default: 20, max: 100).
- `cursor`: The ID of the last item from the previous page (for `after`) or first item (for `before`).

```
GET /api/v1/ads?limit=25&after=cursor-from-last-item
```

## Response

The response **must** include the `data` array and a `meta` object with pagination info.

```json
{
  "data": [
    // ... array of resources
  ],
  "meta": {
    "hasNextPage": true,
    "hasPreviousPage": false,
    "startCursor": "cursor-of-first-item",
    "endCursor": "cursor-of-last-item"
  }
}
```

## Implementation (TypeORM)

```typescript
async function paginate<T>(
  queryBuilder: SelectQueryBuilder<T>,
  { limit = 20, after, before }: { limit?: number; after?: string; before?: string }
) {
  if (after) {
    queryBuilder.where('entity.id > :after', { after });
  }
  if (before) {
    queryBuilder.where('entity.id < :before', { before });
  }

  const items = await queryBuilder
    .orderBy('entity.id', before ? 'DESC' : 'ASC')
    .take(limit + 1) // Fetch one extra to check for next page
    .getMany();

  const hasMore = items.length > limit;
  if (hasMore) {
    items.pop(); // Remove the extra item
  }

  if (before) {
    items.reverse(); // Correct order when fetching previous page
  }

  return {
    data: items,
    meta: {
      hasNextPage: before ? true : hasMore,
      hasPreviousPage: after ? true : (before ? hasMore : false),
      startCursor: items.length > 0 ? items[0].id : null,
      endCursor: items.length > 0 ? items[items.length - 1].id : null,
    },
  };
}
```
