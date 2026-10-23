[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / totpEnrollFinish

# Function: totpEnrollFinish()

> **totpEnrollFinish**(`code`): `Promise`\<`void`\>

Defined in: [api.ts:1512](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1512)

Confirms the pending TOTP secret with an authenticator code.

## Parameters

### code

`string`

Six-digit code from the authenticator app.

## Returns

`Promise`\<`void`\>

Resolves when enrollment is confirmed.
