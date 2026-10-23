[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvGetOk

# Interface: KvGetOk

Defined in: [generated.ts:2339](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2339)

Success payload of `KvNamespace.get`.

## Properties

### expirationUnixMs

> **expirationUnixMs**: `number`

Defined in: [generated.ts:2347](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2347)

Absolute expiry (UTC Unix milliseconds); zero means none.

***

### found

> **found**: `boolean`

Defined in: [generated.ts:2341](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2341)

True when the key exists and has not expired.

***

### metadata

> **metadata**: `string`

Defined in: [generated.ts:2345](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2345)

Stored metadata JSON text; empty when none.

***

### value

> **value**: `Uint8Array`

Defined in: [generated.ts:2343](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2343)

Stored value; empty when not found.
