[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [libraryFilters](../README.md) / bookMatchesFilter

# Function: bookMatchesFilter()

> **bookMatchesFilter**(`book`, `filter`): `boolean`

Defined in: [libraryFilters.ts:158](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/libraryFilters.ts#L158)

Returns whether a book matches the active library filter.

## Parameters

### book

[`BookRecord`](../../api/interfaces/BookRecord.md)

Library row.

### filter

[`LibraryFilter`](../interfaces/LibraryFilter.md)

Kind + value (`all` / empty value always matches).

## Returns

`boolean`

True when the book should remain visible.
