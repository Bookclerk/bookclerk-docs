[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / queueBatchResults

# Function: queueBatchResults()

> **queueBatchResults**(`batch`, `failed`): [`QueueOutcome`](../type-aliases/QueueOutcome.md)[]

Defined in: [plugin.ts:1168](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1168)

Translate recorded [QueueMessage](../classes/QueueMessage.md) outcomes into the wire list.
`failed` is the error thrown by the author's `queue()`; undecided messages
then retry instead of ack.

## Parameters

### batch

[`QueueBatch`](../classes/QueueBatch.md)

Delivered batch after the handler ran.

### failed

`unknown`

Error thrown by the handler, if any.

## Returns

[`QueueOutcome`](../type-aliases/QueueOutcome.md)[]

One outcome per message, in order.
