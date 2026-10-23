[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / bootstrapAdministrator

# Function: bootstrapAdministrator()

> **bootstrapAdministrator**(`body`): `Promise`\<[`BootstrapAdministratorResponse`](../interfaces/BootstrapAdministratorResponse.md)\>

Defined in: [api.ts:1210](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1210)

Creates the first Owner account when no owners (or legacy administrators) exist.

## Parameters

### body

Optional display/login/email/password fields for the new owner.

#### display_name?

`string`

#### email?

`string`

#### login_name?

`string`

#### password?

`string`

## Returns

`Promise`\<[`BootstrapAdministratorResponse`](../interfaces/BootstrapAdministratorResponse.md)\>

Bootstrap result including the one-time invite magic link.
