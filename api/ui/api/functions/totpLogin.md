[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / totpLogin

# Function: totpLogin()

> **totpLogin**(`challengeId`, `code`): `Promise`\<`void`\>

Defined in: [api.ts:1464](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1464)

Completes password+TOTP login via `POST /api/auth/totp/login`.

## Parameters

### challengeId

`string`

Challenge from the password-login MFA payload.

### code

`string`

Six-digit authenticator code.

## Returns

`Promise`\<`void`\>

Resolves when the session cookie is set.
