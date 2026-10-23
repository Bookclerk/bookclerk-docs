[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvListOptions

# Interface: KvListOptions

Defined in: [generated.ts:2378](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2378)

Page request for `KvNamespace.list`.

## Properties

### cursor

> **cursor**: `string`

Defined in: [generated.ts:2382](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2382)

Opaque cursor from a previous page; empty means the first page.

***

### limit

> **limit**: `number`

Defined in: [generated.ts:2384](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2384)

Most keys to return; zero means the host default, at most `maxListPage`.

***

### prefix

> **prefix**: `string`

Defined in: [generated.ts:2380](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2380)

Only keys starting with this prefix; empty means all.
