[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueBatchOk

# Interface: QueueBatchOk

Defined in: [generated.ts:2294](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2294)

Success payload of `QueueConsumer.queue`: exactly one entry per
`QueueBatch.messages` entry, in order. A short list is a host-side error
and the missing tail is retried.

## Properties

### results

> **results**: [`QueueResult`](../type-aliases/QueueResult.md)[]

Defined in: [generated.ts:2296](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2296)

Per-message outcomes in delivery order.
