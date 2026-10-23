[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / setPassword

# Function: setPassword()

> **setPassword**(`body`): `Promise`\<\{ `ok`: `boolean`; `revoked_sessions`: `number`; \}\>

Defined in: [api.ts:1365](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1365)

Sets a password for self or for a managed user.

## Parameters

### body

New password and optional target user id.

#### current_password?

`string`

#### password

`string`

#### user_id?

`number`

## Returns

`Promise`\<\{ `ok`: `boolean`; `revoked_sessions`: `number`; \}\>

Count of revoked sessions.
