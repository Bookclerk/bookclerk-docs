[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchTitleMetaBatch

# Function: fetchTitleMetaBatch()

> **fetchTitleMetaBatch**(`queries`): `Promise`\<([`TitleMeta`](../interfaces/TitleMeta.md) \| `null`)[]\>

Defined in: [api.ts:3601](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3601)

Batch title-meta fetch (order preserved); shares the client TTL cache with singles.

## Parameters

### queries

[`TitleMetaQuery`](../type-aliases/TitleMetaQuery.md)[]

Parallel title-meta queries.

## Returns

`Promise`\<([`TitleMeta`](../interfaces/TitleMeta.md) \| `null`)[]\>

Metadata (or `null`) aligned with `queries`.
