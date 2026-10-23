[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvListResultBinding

# Interface: KvListResultBinding

Defined in: [env.ts:115](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L115)

One page of [KvNamespaceBinding.list](KvNamespaceBinding.md#list).

## Properties

### cursor?

> `optional` **cursor?**: `string`

Defined in: [env.ts:121](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L121)

Cursor for the next page; absent when `list_complete`.

***

### keys

> **keys**: [`KvListKeyBinding`](KvListKeyBinding.md)[]

Defined in: [env.ts:117](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L117)

Keys in lexical order.

***

### list\_complete

> **list\_complete**: `boolean`

Defined in: [env.ts:119](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L119)

True when no further page exists.
