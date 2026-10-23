[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / NamedDatabase

# Interface: NamedDatabase

Defined in: [plugin.ts:563](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L563)

One named `[[databases]]` binding on the granted context: either an
in-isolate [DatabaseBinding](DatabaseBinding.md) or the adapter's granted-channel
[GrantedDatabaseTransport](GrantedDatabaseTransport.md) the author-side SDK wraps.

## Properties

### database?

> `optional` **database?**: [`DatabaseBinding`](DatabaseBinding.md)

Defined in: [plugin.ts:567](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L567)

Ready binding (same isolate).

***

### name

> **name**: `string`

Defined in: [plugin.ts:565](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L565)

Binding name (`env.<name>`).

***

### transport?

> `optional` **transport?**: [`GrantedDatabaseTransport`](GrantedDatabaseTransport.md)

Defined in: [plugin.ts:569](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L569)

Granted transport (crosses the adapter → author RPC boundary).
