[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueMessage

# Class: QueueMessage

Defined in: [plugin.ts:1036](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1036)

One delivered queue message (`QueueMessage` struct), Workers `Message`-shaped.

Exactly one outcome is recorded per message; the first of `ack` / `retry`
wins and later calls are ignored. Messages left untouched are acked when
`queue()` resolves and retried when it throws.

## Constructors

### Constructor

> **new QueueMessage**(`message`): `QueueMessage`

Defined in: [plugin.ts:1054](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1054)

#### Parameters

##### message

`Partial`\<`QueueMessage`\> \| `undefined`

#### Returns

`QueueMessage`

## Properties

### attempts

> `readonly` **attempts**: `number`

Defined in: [plugin.ts:1048](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1048)

Delivery counter, starting at 1.

***

### body

> `readonly` **body**: `Uint8Array`

Defined in: [plugin.ts:1044](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1044)

Encoded body bytes.

***

### contentType

> `readonly` **contentType**: `string`

Defined in: [plugin.ts:1046](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1046)

Body encoding: `json`, `text`, or `bytes`.

***

### id

> `readonly` **id**: `string`

Defined in: [plugin.ts:1040](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1040)

Outbox row id; stable across redeliveries.

***

### raw

> `readonly` **raw**: `Partial`\<`WireQueueMessage`\>

Defined in: [plugin.ts:1052](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1052)

Raw wire envelope.

***

### source

> `readonly` **source**: `string`

Defined in: [plugin.ts:1050](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1050)

Producer plugin id; empty when the host sent it.

***

### timestamp

> `readonly` **timestamp**: `Date`

Defined in: [plugin.ts:1042](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1042)

When the producer sent the message.

## Accessors

### result

#### Get Signature

> **get** **result**(): [`QueueOutcome`](../type-aliases/QueueOutcome.md) \| `null`

Defined in: [plugin.ts:1095](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1095)

Recorded outcome.

##### Returns

[`QueueOutcome`](../type-aliases/QueueOutcome.md) \| `null`

The outcome, or `null` while undecided.

## Methods

### ack()

> **ack**(): `void`

Defined in: [plugin.ts:1104](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1104)

Mark handled; the host marks the message delivered.

#### Returns

`void`

***

### json()

> **json**\<`T`\>(): `T`

Defined in: [plugin.ts:1076](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1076)

Decode [QueueMessage.body](#body) as JSON.

#### Type Parameters

##### T

`T` = `unknown`

#### Returns

`T`

Parsed body (`null` for an empty body).

***

### retry()

> **retry**(`options?`): `void`

Defined in: [plugin.ts:1114](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1114)

Redeliver later: after `delaySeconds`, or — when omitted — after the
consumer's `retry_delay`.

#### Parameters

##### options?

[`QueueRetryOptions`](../interfaces/QueueRetryOptions.md)

Redelivery timing and reason.

#### Returns

`void`

***

### text()

> **text**(): `string`

Defined in: [plugin.ts:1086](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1086)

Decode [QueueMessage.body](#body) as UTF-8 text.

#### Returns

`string`

The body text.
