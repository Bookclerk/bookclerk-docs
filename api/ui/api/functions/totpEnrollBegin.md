[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / totpEnrollBegin

# Function: totpEnrollBegin()

> **totpEnrollBegin**(`currentPassword?`): `Promise`\<\{ `otpauth_url`: `string`; `qr_svg`: `string`; `secret`: `string`; \}\>

Defined in: [api.ts:1490](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1490)

Starts TOTP enrollment (pending secret, otpauth URL, QR SVG).

## Parameters

### currentPassword?

`string`

Required when the portal session is older than the reauth window.

## Returns

`Promise`\<\{ `otpauth_url`: `string`; `qr_svg`: `string`; `secret`: `string`; \}\>

Pending secret, otpauth URL, and QR SVG.
