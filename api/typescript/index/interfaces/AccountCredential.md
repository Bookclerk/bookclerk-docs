[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / AccountCredential

# Interface: AccountCredential

Defined in: [generated.ts:1488](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1488)

Host-sealed credentials for one account, delivered on `scan`.

## Properties

### accountId

> **accountId**: `string`

Defined in: [generated.ts:1490](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1490)

Account id the blob belongs to.

***

### credentials

> **credentials**: `Uint8Array`

Defined in: [generated.ts:1492](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1492)

Opaque credential bytes exactly as the guest returned them at login.
