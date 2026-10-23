[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / applyTitleMeta

# Function: applyTitleMeta()

> **applyTitleMeta**(`title`, `meta`): [`CatalogTitle`](../type-aliases/CatalogTitle.md)

Defined in: [catalogTitle.ts:923](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L923)

Overlays public Audnexus / catalog metadata onto a sparse [CatalogTitle](../type-aliases/CatalogTitle.md).

An Audnexus ASIN is a trusted Audible product id (same rule as purchase-hint
resolve), so a missing Audible store edition is filled in for Where-to-buy.

## Parameters

### title

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Existing title.

### meta

[`TitleMeta`](../../api/interfaces/TitleMeta.md) \| `null` \| `undefined`

Fetched metadata (no-op when nullish).

## Returns

[`CatalogTitle`](../type-aliases/CatalogTitle.md)

Merged title.
