[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ScalarLimits

# Interface: ScalarLimits

Defined in: [generated.ts:49](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L49)

Guest-advertised caps for `rpc.scalarLimits`; never above the file constants.

## Properties

### maxListPage

> **maxListPage**: `number`

Defined in: [generated.ts:55](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L55)

Largest list page; at most `maxListPage`.

***

### maxScalarBytes

> **maxScalarBytes**: `number`

Defined in: [generated.ts:51](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L51)

Largest scalar the guest accepts; at most `maxScalarBytes`.

***

### maxStreamWindowBytes

> **maxStreamWindowBytes**: `number`

Defined in: [generated.ts:53](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L53)

Largest `ByteSource.pull` window; at most `maxStreamWindowBytes`.
