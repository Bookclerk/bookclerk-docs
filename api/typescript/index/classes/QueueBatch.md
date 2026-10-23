[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueBatch

# Class: QueueBatch

Defined in: [plugin.ts:1124](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1124)

One `queue(batch)` delivery (`QueueBatch` struct), Workers `MessageBatch`-shaped.

## Constructors

### Constructor

> **new QueueBatch**(`queue`, `messages`, `invocation?`): `QueueBatch`

Defined in: [plugin.ts:1132](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1132)

#### Parameters

##### queue

`string` \| `undefined`

##### messages

readonly `Partial`\<`QueueMessage`\>[] \| `undefined`

##### invocation?

[`Invocation`](../interfaces/Invocation.md)

#### Returns

`QueueBatch`

## Properties

### invocation

> **invocation**: [`Invocation`](../interfaces/Invocation.md)

Defined in: [plugin.ts:1130](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1130)

Invocation identity of this delivery.

***

### messages

> `readonly` **messages**: readonly [`QueueMessage`](QueueMessage.md)[]

Defined in: [plugin.ts:1128](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1128)

Messages in delivery order.

***

### queue

> `readonly` **queue**: `string`

Defined in: [plugin.ts:1126](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1126)

Queue the messages were sent to.

## Methods

### ackAll()

> **ackAll**(): `void`

Defined in: [plugin.ts:1145](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1145)

Ack every message.

#### Returns

`void`

***

### retryAll()

> **retryAll**(`options?`): `void`

Defined in: [plugin.ts:1154](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1154)

Retry every message.

#### Parameters

##### options?

[`QueueRetryOptions`](../interfaces/QueueRetryOptions.md)

Redelivery timing and reason.

#### Returns

`void`
