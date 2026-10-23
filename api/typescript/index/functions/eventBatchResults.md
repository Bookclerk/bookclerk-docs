[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / eventBatchResults

# Function: eventBatchResults()

> **eventBatchResults**(`batch`, `failed`): [`EventOutcome`](../type-aliases/EventOutcome.md)[]

Defined in: [plugin.ts:1004](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1004)

Translate recorded [EventMessage](../classes/EventMessage.md) outcomes into the wire list.
`failed` is the error thrown by the author's `event()`; undecided messages
then retry instead of ack.

## Parameters

### batch

[`EventBatch`](../classes/EventBatch.md)

Delivered batch after the handler ran.

### failed

`unknown`

Error thrown by the handler, if any.

## Returns

[`EventOutcome`](../type-aliases/EventOutcome.md)[]

One outcome per message, in order.
