[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [libraryFilters](../README.md) / parseSeriesIndex

# Function: parseSeriesIndex()

> **parseSeriesIndex**(`raw`): `number` \| `null`

Defined in: [libraryFilters.ts:202](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/libraryFilters.ts#L202)

Parses a series position like `3`, `#3`, `3.5`, or `Book 3` for numeric sort.

## Parameters

### raw

`string` \| `null` \| `undefined`

Series index string from the store.

## Returns

`number` \| `null`

Parsed number, or `null` when absent / non-numeric.
