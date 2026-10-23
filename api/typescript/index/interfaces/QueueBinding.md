[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueBinding

# Interface: QueueBinding

Defined in: [env.ts:59](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L59)

`[[queues.producers]]` binding (`env.<BINDING>`): Workers `Queue`-shaped
producer into the queue the row names. The host assigns ids, stamps the
sending plugin as `source`, and bounds each body at `maxEventPayloadBytes`.

## Methods

### send()

> **send**(`body`, `options?`): `Promise`\<[`QueueSendOk`](QueueSendOk.md)\>

Defined in: [env.ts:67](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L67)

Send one message.

#### Parameters

##### body

`unknown`

JSON value (default), string, or bytes per `contentType`.

##### options?

[`QueueSendOptions`](QueueSendOptions.md)

Encoding and delay.

#### Returns

`Promise`\<[`QueueSendOk`](QueueSendOk.md)\>

The outbox id the host assigned.

***

### sendBatch()

> **sendBatch**(`messages`, `options?`): `Promise`\<[`QueueSendOk`](QueueSendOk.md)\>

Defined in: [env.ts:75](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L75)

Send up to `maxListPage` messages in order.

#### Parameters

##### messages

readonly [`QueueSendBatchEntry`](QueueSendBatchEntry.md)[]

Bodies with per-message options.

##### options?

`Pick`\<[`QueueSendOptions`](QueueSendOptions.md), `"delaySeconds"`\>

Default delay for entries that carry none.

#### Returns

`Promise`\<[`QueueSendOk`](QueueSendOk.md)\>

The outbox ids in send order.
