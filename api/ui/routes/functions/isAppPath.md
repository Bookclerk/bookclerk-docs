[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / isAppPath

# Function: isAppPath()

> **isAppPath**(`pathname`): `boolean`

Defined in: [routes.ts:50](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L50)

Returns whether `pathname` is a known GUI route (not an API/static/unknown URL).

## Parameters

### pathname

`string`

Document pathname (may include a trailing slash).

## Returns

`boolean`

True when the path is owned by the React SPA.
