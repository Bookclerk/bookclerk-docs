[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [libraryFilters](../README.md) / sortBooks

# Function: sortBooks()

> **sortBooks**(`books`, `sort`): [`BookRecord`](../../api/interfaces/BookRecord.md)[]

Defined in: [libraryFilters.ts:239](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/libraryFilters.ts#L239)

Returns a new array of books sorted by the given key.

## Parameters

### books

[`BookRecord`](../../api/interfaces/BookRecord.md)[]

Library rows.

### sort

[`SortKey`](../type-aliases/SortKey.md)

Sort key.

## Returns

[`BookRecord`](../../api/interfaces/BookRecord.md)[]

Sorted copy (stable enough for UI tables).
