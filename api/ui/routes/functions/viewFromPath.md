[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / viewFromPath

# Function: viewFromPath()

> **viewFromPath**(`pathname`): [`AppView`](../../api/type-aliases/AppView.md) \| `null`

Defined in: [routes.ts:74](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L74)

Maps a document path to an app view, or `null` for `/` (use default_view).

## Parameters

### pathname

`string`

Document pathname.

## Returns

[`AppView`](../../api/type-aliases/AppView.md) \| `null`

Matching [AppView](../../api/type-aliases/AppView.md), or `null` when the path is `/` or unknown.
