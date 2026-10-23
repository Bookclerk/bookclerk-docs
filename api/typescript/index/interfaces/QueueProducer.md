[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueProducer

# Interface: QueueProducer

Defined in: [generated.ts:876](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L876)

Queue producer binding: host-served send into the queue a
`[[queues.producers]]` row names, granted through `Bindings.queues`.

## Methods

### send()

> **send**(`message`): `Promise`\<[`QueueSendReply`](../type-aliases/QueueSendReply.md)\>

Defined in: [generated.ts:883](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L883)

Send one message.

#### Parameters

##### message

[`QueueSendMessage`](QueueSendMessage.md)

Body, encoding, and delay.

#### Returns

`Promise`\<[`QueueSendReply`](../type-aliases/QueueSendReply.md)\>

[QueueSendReply](../type-aliases/QueueSendReply.md)

***

### sendBatch()

> **sendBatch**(`messages`, `delaySeconds`): `Promise`\<[`QueueSendReply`](../type-aliases/QueueSendReply.md)\>

Defined in: [generated.ts:892](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L892)

Send up to `maxListPage` messages in order; `delaySeconds` applies to
every message that carries no delay of its own.

#### Parameters

##### messages

[`QueueSendMessage`](QueueSendMessage.md)[]

Messages in send order.

##### delaySeconds

`number`

Default delay for the batch.

#### Returns

`Promise`\<[`QueueSendReply`](../type-aliases/QueueSendReply.md)\>

[QueueSendReply](../type-aliases/QueueSendReply.md)
