[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / resetUserPassword

# Function: resetUserPassword()

> **resetUserPassword**(`id`): `Promise`\<\{ `claim_ticket`: `string`; `invite_url?`: `string` \| `null`; `ok`: `boolean`; `revoked_sessions`: `number`; \}\>

Defined in: [api.ts:1300](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1300)

Mints a password-reset claim ticket and revokes the user's portal sessions.

## Parameters

### id

`number`

User id.

## Returns

`Promise`\<\{ `claim_ticket`: `string`; `invite_url?`: `string` \| `null`; `ok`: `boolean`; `revoked_sessions`: `number`; \}\>

Reset ticket plus count of revoked sessions.
