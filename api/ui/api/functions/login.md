[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / login

# Function: login()

> **login**(`token`): `Promise`\<[`AuthSession`](../interfaces/AuthSession.md)\>

Defined in: [api.ts:550](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L550)

Signs in with the operator token via `POST /api/auth/login`.

## Parameters

### token

`string`

Operator bearer token from `bookclerk daemon token`.

## Returns

`Promise`\<[`AuthSession`](../interfaces/AuthSession.md)\>

Authenticated operator session (acquire enabled).
