[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventMessage

# Class: EventMessage

Defined in: [plugin.ts:805](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L805)

One delivered domain event. Exactly one outcome is recorded per message;
the first of `ack` / `retry` / `reject` / `deadLetter` / `suspend` wins and
later calls are ignored (as with Workers queue messages). Messages left
untouched are acked when `event()` resolves and retried when it throws.

## Constructors

### Constructor

> **new EventMessage**(`event`): `EventMessage`

Defined in: [plugin.ts:839](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L839)

#### Parameters

##### event

`Partial`\<[`DomainEvent`](../interfaces/DomainEvent.md)\> \| `undefined`

#### Returns

`EventMessage`

## Properties

### accountId

> `readonly` **accountId**: `string`

Defined in: [plugin.ts:819](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L819)

Account scope; empty for host-wide events.

***

### attempts

> `readonly` **attempts**: `number`

Defined in: [plugin.ts:817](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L817)

Delivery counter, starting at 1.

***

### body

> `readonly` **body**: `Uint8Array`

Defined in: [plugin.ts:829](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L829)

Encoded payload bytes.

***

### causationId

> `readonly` **causationId**: `string`

Defined in: [plugin.ts:825](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L825)

Id of the command or event that caused this one.

***

### checkpoint

> `readonly` **checkpoint**: [`DeliveredCheckpoint`](../interfaces/DeliveredCheckpoint.md) \| `null`

Defined in: [plugin.ts:835](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L835)

Checkpoint from the prior `suspend()`, or `null`.

***

### correlationId

> `readonly` **correlationId**: `string`

Defined in: [plugin.ts:823](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L823)

Trace correlation id.

***

### deduplicationKey

> `readonly` **deduplicationKey**: `string`

Defined in: [plugin.ts:827](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L827)

Consumer-side idempotency key; stable across redeliveries.

***

### id

> `readonly` **id**: `string`

Defined in: [plugin.ts:809](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L809)

Outbox event id.

***

### invocationSequence

> `readonly` **invocationSequence**: `number`

Defined in: [plugin.ts:831](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L831)

Resume ordinal; distinct from `attempts`.

***

### raw

> `readonly` **raw**: `Partial`\<[`DomainEvent`](../interfaces/DomainEvent.md)\>

Defined in: [plugin.ts:837](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L837)

Raw wire envelope.

***

### resumePending

> `readonly` **resumePending**: `boolean`

Defined in: [plugin.ts:833](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L833)

True when this delivery resumes a prior `suspend()`.

***

### schemaVersion

> `readonly` **schemaVersion**: `number`

Defined in: [plugin.ts:813](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L813)

Schema version of [EventMessage.body](#body).

***

### source

> `readonly` **source**: `string`

Defined in: [plugin.ts:821](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L821)

Producer plugin id; empty when unknown.

***

### timestamp

> `readonly` **timestamp**: `Date`

Defined in: [plugin.ts:815](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L815)

When the producer observed the fact.

***

### type

> `readonly` **type**: `string`

Defined in: [plugin.ts:811](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L811)

Event type (`book_acquired`, …).

## Accessors

### result

#### Get Signature

> **get** **result**(): [`EventOutcome`](../type-aliases/EventOutcome.md) \| `null`

Defined in: [plugin.ts:885](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L885)

Recorded outcome.

##### Returns

[`EventOutcome`](../type-aliases/EventOutcome.md) \| `null`

The outcome, or `null` while undecided.

## Methods

### ack()

> **ack**(): `void`

Defined in: [plugin.ts:894](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L894)

Mark handled; the host marks the event delivered.

#### Returns

`void`

***

### deadLetter()

> **deadLetter**(`reason?`): `void`

Defined in: [plugin.ts:928](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L928)

Park for operator review.

#### Parameters

##### reason?

`string`

Short operator-facing reason.

#### Returns

`void`

***

### json()

> **json**\<`T`\>(): `T`

Defined in: [plugin.ts:875](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L875)

Decode [EventMessage.body](#body) as JSON.

#### Type Parameters

##### T

`T` = [`JsonObject`](../type-aliases/JsonObject.md)

#### Returns

`T`

Parsed payload (`{}` for an empty body).

***

### reject()

> **reject**(`reason?`): `void`

Defined in: [plugin.ts:919](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L919)

Stop delivering; the host records `reason`.

#### Parameters

##### reason?

`string`

Short operator-facing reason.

#### Returns

`void`

***

### retry()

> **retry**(`options?`): `void`

Defined in: [plugin.ts:904](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L904)

Redeliver later: at `retryAt`, after `delaySeconds`, or — when both are
omitted — whenever the host's backoff chooses.

#### Parameters

##### options?

[`RetryOptions`](../interfaces/RetryOptions.md)

Redelivery timing and reason.

#### Returns

`void`

***

### suspend()

> **suspend**(`options?`): `void`

Defined in: [plugin.ts:938](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L938)

Release with a bounded checkpoint; the host redelivers at `wakeAt` or
when a `wakeOnEventType` event arrives.

#### Parameters

##### options?

[`EventSuspendOptions`](../interfaces/EventSuspendOptions.md)

Checkpoint and wake conditions.

#### Returns

`void`
