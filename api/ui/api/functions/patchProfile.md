[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / patchProfile

# Function: patchProfile()

> **patchProfile**(`body`): `Promise`\<\{ `ok`: `boolean`; `user`: [`AuthMeUser`](../interfaces/AuthMeUser.md); \}\>

Defined in: [api.ts:1385](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1385)

Updates the signed-in user's display name and/or email.

## Parameters

### body

Fields to change; omitted keys are left unchanged.

#### avatar_source?

`string`

#### display_name?

`string`

#### email?

`string`

## Returns

`Promise`\<\{ `ok`: `boolean`; `user`: [`AuthMeUser`](../interfaces/AuthMeUser.md); \}\>

Updated profile snapshot from the daemon.
