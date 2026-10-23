[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvListOk

# Interface: KvListOk

Defined in: [generated.ts:2398](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2398)

Success payload of `KvNamespace.list`.

## Properties

### cursor

> **cursor**: `string`

Defined in: [generated.ts:2404](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2404)

Cursor for the next page; empty when `listComplete`.

***

### keys

> **keys**: [`KvListKey`](KvListKey.md)[]

Defined in: [generated.ts:2400](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2400)

Keys in lexical order.

***

### listComplete

> **listComplete**: `boolean`

Defined in: [generated.ts:2402](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2402)

True when no further page exists.
