[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / isInvitePath

# Function: isInvitePath()

> **isInvitePath**(`pathname?`): `boolean`

Defined in: [routes.ts:156](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L156)

True when this document is the magic-link invite route.

## Parameters

### pathname?

`string` = `window.location.pathname`

Document pathname.

## Returns

`boolean`

True for `/invite` (trailing slash ignored).
