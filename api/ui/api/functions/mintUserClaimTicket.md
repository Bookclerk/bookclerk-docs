[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / mintUserClaimTicket

# Function: mintUserClaimTicket()

> **mintUserClaimTicket**(`id`): `Promise`\<\{ `claim_ticket`: `string`; `invite_url?`: `string` \| `null`; `ok`: `boolean`; \}\>

Defined in: [api.ts:1280](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1280)

Mints a fresh one-time claim ticket for an existing active user.

## Parameters

### id

`number`

User id.

## Returns

`Promise`\<\{ `claim_ticket`: `string`; `invite_url?`: `string` \| `null`; `ok`: `boolean`; \}\>

One-time claim ticket.
