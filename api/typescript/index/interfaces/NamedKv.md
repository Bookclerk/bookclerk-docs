[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / NamedKv

# Interface: NamedKv

Defined in: [plugin.ts:526](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L526)

One granted `[[kv_namespaces]]` binding on the granted context.

## Properties

### kv

> **kv**: [`KvNamespaceBinding`](KvNamespaceBinding.md)

Defined in: [plugin.ts:530](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L530)

Ready namespace (crosses the adapter → author RPC boundary).

***

### name

> **name**: `string`

Defined in: [plugin.ts:528](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L528)

Binding name (`env.<name>`).
