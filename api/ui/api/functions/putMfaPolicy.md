[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / putMfaPolicy

# Function: putMfaPolicy()

> **putMfaPolicy**(`requireSecondFactor`, `currentPassword?`): `Promise`\<\{ `require_second_factor`: `boolean`; \}\>

Defined in: [api.ts:1557](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1557)

Persists whether password login requires TOTP or a passkey.

## Parameters

### requireSecondFactor

`boolean`

When true, password-only sign-in is not enough once enrolled.

### currentPassword?

`string`

Owner step-up; operators omit this.

## Returns

`Promise`\<\{ `require_second_factor`: `boolean`; \}\>

Saved `{ require_second_factor }` flag.
