[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventConsumer

# Interface: EventConsumer

Defined in: [generated.ts:823](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L823)

`[[events.consumers]]` handler (`Entrypoints.eventConsumer`). Delivery is
at-least-once; consumers must be idempotent on `deduplicationKey`.

## Methods

### event()

> **event**(`batch`): `Promise`\<[`EventBatchReply`](../type-aliases/EventBatchReply.md)\>

Defined in: [generated.ts:830](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L830)

Deliver one ordered batch of domain events.

#### Parameters

##### batch

`EventBatch`

Events in delivery order.

#### Returns

`Promise`\<[`EventBatchReply`](../type-aliases/EventBatchReply.md)\>

[EventBatchReply](../type-aliases/EventBatchReply.md)
