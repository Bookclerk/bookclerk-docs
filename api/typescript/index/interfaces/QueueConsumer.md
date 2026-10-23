[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueConsumer

# Interface: QueueConsumer

Defined in: [generated.ts:862](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L862)

`[[queues.consumers]]` handler (`Entrypoints.queueConsumer`). Delivery is
at-least-once; consumers must be idempotent on `QueueMessage.id`.

## Methods

### queue()

> **queue**(`batch`): `Promise`\<[`QueueBatchReply`](../type-aliases/QueueBatchReply.md)\>

Defined in: [generated.ts:869](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L869)

Deliver one ordered batch of messages from one queue.

#### Parameters

##### batch

`QueueBatch`

Messages in delivery order.

#### Returns

`Promise`\<[`QueueBatchReply`](../type-aliases/QueueBatchReply.md)\>

[QueueBatchReply](../type-aliases/QueueBatchReply.md)
