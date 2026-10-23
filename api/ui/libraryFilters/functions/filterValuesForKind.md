[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [libraryFilters](../README.md) / filterValuesForKind

# Function: filterValuesForKind()

> **filterValuesForKind**(`books`, `kind`): `object`[]

Defined in: [libraryFilters.ts:92](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/libraryFilters.ts#L92)

Builds distinct filter value options for a kind from the loaded books.

## Parameters

### books

[`BookRecord`](../../api/interfaces/BookRecord.md)[]

Library rows to mine for facet values.

### kind

[`FilterKind`](../type-aliases/FilterKind.md)

Filter category.

## Returns

`object`[]

Value/label pairs (empty for `all`).
