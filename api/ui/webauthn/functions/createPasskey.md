[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [webauthn](../README.md) / createPasskey

# Function: createPasskey()

> **createPasskey**(`options`): `Promise`\<\{ `challenge_id`: `string`; `credential`: `Record`\<`string`, `unknown`\>; \}\>

Defined in: [webauthn.ts:223](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/webauthn.ts#L223)

Run a registration ceremony and return JSON for `/register/finish`.

## Parameters

### options

[`PasskeyCeremonyOptions`](../interfaces/PasskeyCeremonyOptions.md)

## Returns

`Promise`\<\{ `challenge_id`: `string`; `credential`: `Record`\<`string`, `unknown`\>; \}\>
