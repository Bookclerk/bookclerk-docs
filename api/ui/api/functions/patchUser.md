[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / patchUser

# Function: patchUser()

> **patchUser**(`id`, `body`): `Promise`\<[`UserMutationResponse`](../interfaces/UserMutationResponse.md)\>

Defined in: [api.ts:1255](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1255)

Updates a first-party user role, status, or profile fields.

## Parameters

### id

`number`

User id to patch.

### body

Partial user fields accepted by the daemon.

#### display_name?

`string`

#### email?

`string`

#### login_name?

`string`

#### role?

`string`

#### status?

`string`

## Returns

`Promise`\<[`UserMutationResponse`](../interfaces/UserMutationResponse.md)\>

Patched user row.
