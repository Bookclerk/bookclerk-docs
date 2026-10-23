[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / storeLabel

# Function: storeLabel()

> **storeLabel**(`source`): `string`

Defined in: [catalogTitle.ts:310](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L310)

Human-readable label for a storefront plugin id in Discover filters.

Falls back to the raw id when the store is unknown so future plugins still render.

## Parameters

### source

`string`

Store id (`audible`, `libro`, …).

## Returns

`string`

Display name for UI chrome.
