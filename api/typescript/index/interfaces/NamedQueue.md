[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / NamedQueue

# Interface: NamedQueue

Defined in: [plugin.ts:534](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L534)

One granted `[[queues.producers]]` binding on the granted context.

## Properties

### name

> **name**: `string`

Defined in: [plugin.ts:536](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L536)

Binding name (`env.<name>`).

***

### queue

> **queue**: [`QueueBinding`](QueueBinding.md)

Defined in: [plugin.ts:538](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L538)

Ready producer (crosses the adapter → author RPC boundary).
