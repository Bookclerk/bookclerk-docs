[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / createUser

# Function: createUser()

> **createUser**(`body`): `Promise`\<[`UserMutationResponse`](../interfaces/UserMutationResponse.md)\>

Defined in: [api.ts:1231](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1231)

Creates a first-party Bookclerk user.

## Parameters

### body

Role, profile fields, optional password, and invite choice.

#### display_name?

`string`

#### email?

`string`

#### login_name?

`string`

#### mint_invite?

`boolean`

#### password?

`string`

#### role?

`string`

## Returns

`Promise`\<[`UserMutationResponse`](../interfaces/UserMutationResponse.md)\>

Created user plus optional one-time claim ticket.
