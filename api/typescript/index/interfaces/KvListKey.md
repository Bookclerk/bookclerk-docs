[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvListKey

# Interface: KvListKey

Defined in: [generated.ts:2388](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2388)

One key on a `KvNamespace.list` page.

## Properties

### expirationUnixMs

> **expirationUnixMs**: `number`

Defined in: [generated.ts:2394](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2394)

Absolute expiry (UTC Unix milliseconds); zero means none.

***

### key

> **key**: `string`

Defined in: [generated.ts:2390](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2390)

Key name.

***

### metadata

> **metadata**: `string`

Defined in: [generated.ts:2392](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2392)

Stored metadata JSON text; empty when none.
