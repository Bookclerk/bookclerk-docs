[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchTitleMeta

# Function: fetchTitleMeta()

> **fetchTitleMeta**(`body`): `Promise`\<[`TitleMeta`](../interfaces/TitleMeta.md) \| `null`\>

Defined in: [api.ts:3548](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3548)

Fetches title metadata, coalescing viewport-gated cards into a short batch window.

## Parameters

### body

[`TitleMetaQuery`](../type-aliases/TitleMetaQuery.md)

Title identity fields.

## Returns

`Promise`\<[`TitleMeta`](../interfaces/TitleMeta.md) \| `null`\>

Metadata or `null` when none matched (client-cached).
