[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / normalizeAppView

# Function: normalizeAppView()

> **normalizeAppView**(`v`): [`AppView`](../../api/type-aliases/AppView.md)

Defined in: [routes.ts:137](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L137)

Coerces an unknown string into a valid [AppView](../../api/type-aliases/AppView.md).

## Parameters

### v

`string` \| `undefined`

Candidate view id from the session or URL.

## Returns

[`AppView`](../../api/type-aliases/AppView.md)

Valid view, defaulting to `discover`.
