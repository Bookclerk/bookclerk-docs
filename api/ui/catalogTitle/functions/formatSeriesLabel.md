[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / formatSeriesLabel

# Function: formatSeriesLabel()

> **formatSeriesLabel**(`series`, `seriesIndex?`): `string` \| `null`

Defined in: [catalogTitle.ts:471](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L471)

Formats series name + book number for list/card rows (`Sun Eater #3`).

## Parameters

### series

`string` \| `null` \| `undefined`

Series name.

### seriesIndex?

`string` \| `null`

Optional book number.

## Returns

`string` \| `null`

Label, or `null` when series is empty.
