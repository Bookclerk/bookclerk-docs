[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / syncUrlToView

# Function: syncUrlToView()

> **syncUrlToView**(`view`, `mode?`): `void`

Defined in: [routes.ts:183](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L183)

Updates the URL bar without a full reload.

Drops `ticket` and `sso_error` so a claimed invite URL is not kept on
`/discover?ticket=…` (a refresh of that path would skip the 410 invite page).

## Parameters

### view

[`AppView`](../../api/type-aliases/AppView.md)

View to reflect in the path.

### mode?

`"replace"` \| `"push"`

History `push` (default) or `replace`.

## Returns

`void`
