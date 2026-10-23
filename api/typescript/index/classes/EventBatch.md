[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventBatch

# Class: EventBatch

Defined in: [plugin.ts:957](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L957)

One `event(batch)` delivery (`EventBatch` struct), Workers `MessageBatch`-shaped.

## Constructors

### Constructor

> **new EventBatch**(`events`, `invocation?`): `EventBatch`

Defined in: [plugin.ts:963](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L963)

#### Parameters

##### events

readonly `Partial`\<[`DomainEvent`](../interfaces/DomainEvent.md)\>[] \| `undefined`

##### invocation?

[`Invocation`](../interfaces/Invocation.md)

#### Returns

`EventBatch`

## Properties

### invocation

> **invocation**: [`Invocation`](../interfaces/Invocation.md)

Defined in: [plugin.ts:961](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L961)

Invocation identity of this delivery.

***

### messages

> `readonly` **messages**: readonly [`EventMessage`](EventMessage.md)[]

Defined in: [plugin.ts:959](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L959)

Messages in delivery order.

## Accessors

### type

#### Get Signature

> **get** **type**(): `string`

Defined in: [plugin.ts:975](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L975)

Event type shared by the batch.

##### Returns

`string`

The shared type, or `""` when mixed or empty.

## Methods

### ackAll()

> **ackAll**(): `void`

Defined in: [plugin.ts:981](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L981)

Ack every message.

#### Returns

`void`

***

### retryAll()

> **retryAll**(`options?`): `void`

Defined in: [plugin.ts:990](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L990)

Retry every message.

#### Parameters

##### options?

[`RetryOptions`](../interfaces/RetryOptions.md)

Redelivery timing and reason.

#### Returns

`void`
