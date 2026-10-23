[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / passwordLogin

# Function: passwordLogin()

> **passwordLogin**(`login`, `password`): `Promise`\<\{ `mfa?`: \{ `challenge_id`: `string`; `method`: `string`; \}; `ok`: `boolean`; \}\>

Defined in: [api.ts:1444](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1444)

Signs in with username/password via `POST /api/auth/password`.

## Parameters

### login

`string`

Username or email.

### password

`string`

Account password.

## Returns

`Promise`\<\{ `mfa?`: \{ `challenge_id`: `string`; `method`: `string`; \}; `ok`: `boolean`; \}\>

Session cookie on success, or an MFA challenge when TOTP is enabled.
