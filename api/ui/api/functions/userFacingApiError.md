[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / userFacingApiError

# Function: userFacingApiError()

> **userFacingApiError**(`error`, `fallback`): `string`

Defined in: [api.ts:363](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L363)

Maps API / network failures to short copy for non-technical users.

Keeps branded daemon messages when they already read as plain language.

## Parameters

### error

`unknown`

Caught failure from fetch / parse helpers.

### fallback

`string`

Copy used when the error has no safe user-facing text.

## Returns

`string`

Message suitable for inline SPA error banners.
