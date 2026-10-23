[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / searchCatalog

# Function: searchCatalog()

> **searchCatalog**(`q`, `opts?`): `Promise`\<[`CatalogSearchPage`](../interfaces/CatalogSearchPage.md)\>

Defined in: [api.ts:3670](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3670)

Searches the Discover catalog across enabled storefronts.

Returns an empty page (without calling the network) when `q` trims to fewer
than 2 characters. Language defaults to the browser locale unless
`allLanguages` is set.

## Parameters

### q

`string`

Free-text query (minimum 2 characters after trim).

### opts?

Pagination cursor, sort, language, and server-side facet filters.

#### allLanguages?

`boolean`

When true, do not hard-filter by language.

#### cursor?

`string` \| `null`

#### field?

`"author"` \| `"narrator"` \| `"series"` \| `"genre"`

#### filters?

[`CatalogSearchFilters`](../type-aliases/CatalogSearchFilters.md)

#### lang?

`string`

#### limit?

`number`

**Deprecated**

use page_size

#### page_size?

`number`

#### sort?

[`CatalogSearchSort`](../type-aliases/CatalogSearchSort.md)

#### sort_dir?

[`CatalogSortDir`](../type-aliases/CatalogSortDir.md)

## Returns

`Promise`\<[`CatalogSearchPage`](../interfaces/CatalogSearchPage.md)\>

Cursor-paginated search page (or an empty stub when `q` is too short).
