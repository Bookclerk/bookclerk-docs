[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / resolveView

# Function: resolveView()

> **resolveView**(`pathname`, `defaultView`): [`AppView`](../../api/type-aliases/AppView.md)

Defined in: [routes.ts:124](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L124)

Resolves the view to show for the current URL plus signed-in default.

## Parameters

### pathname

`string`

Document pathname.

### defaultView

`string` \| `undefined`

Session default view when the path is `/`.

## Returns

[`AppView`](../../api/type-aliases/AppView.md)

Effective [AppView](../../api/type-aliases/AppView.md).
