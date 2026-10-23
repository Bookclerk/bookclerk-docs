[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / KvListOptionsBinding

# Interface: KvListOptionsBinding

Defined in: [env.ts:95](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L95)

Page request for [KvNamespaceBinding.list](KvNamespaceBinding.md#list).

## Properties

### cursor?

> `optional` **cursor?**: `string`

Defined in: [env.ts:99](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L99)

Cursor from a previous page.

***

### limit?

> `optional` **limit?**: `number`

Defined in: [env.ts:101](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L101)

Most keys to return (host default when omitted, at most `maxListPage`).

***

### prefix?

> `optional` **prefix?**: `string`

Defined in: [env.ts:97](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L97)

Only keys starting with this prefix.
