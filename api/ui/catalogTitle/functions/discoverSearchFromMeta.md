[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / discoverSearchFromMeta

# Function: discoverSearchFromMeta()

> **discoverSearchFromMeta**(`kind`, `value`): `object`

Defined in: [catalogTitle.ts:95](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L95)

Composes a Discover catalog search + facet filter from a title-detail meta link.

## Parameters

### kind

`"series"` \| `"authors"` \| `"narrators"` \| `"genres"`

Metadata link kind (authors / narrators / series / genres).

### value

`string`

Linked label text.

## Returns

`object`

Search query, field, sort, and matching include filters.

### authors?

> `optional` **authors?**: `string`[]

### field

> **field**: [`CatalogSearchField`](../type-aliases/CatalogSearchField.md)

### genres?

> `optional` **genres?**: `string`[]

### hideVirtualVoice

> **hideVirtualVoice**: `boolean`

### narrators?

> `optional` **narrators?**: `string`[]

### q

> **q**: `string`

### series?

> `optional` **series?**: `string`[]

### sort

> **sort**: [`CatalogSearchSort`](../../api/type-aliases/CatalogSearchSort.md)
