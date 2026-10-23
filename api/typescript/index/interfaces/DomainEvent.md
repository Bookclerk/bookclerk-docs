[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DomainEvent

# Interface: DomainEvent

Defined in: [generated.ts:442](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L442)

Domain event (not a job). Outbox-produced, at-least-once, idempotent consume.

## Properties

### accountId

> **accountId**: `string`

Defined in: [generated.ts:452](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L452)

Account scope; empty for host-wide events.

***

### causationId

> **causationId**: `string`

Defined in: [generated.ts:456](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L456)

Id of the command or event that caused this one; empty when none.

***

### checkpointJson

> **checkpointJson**: `string`

Defined in: [generated.ts:464](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L464)

Append-only. Resume a prior EventResult.suspended.

***

### checkpointSchemaVersion

> **checkpointSchemaVersion**: `number`

Defined in: [generated.ts:466](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L466)

Schema version of `checkpointJson`.

***

### correlationId

> **correlationId**: `string`

Defined in: [generated.ts:454](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L454)

Trace correlation id; empty when none.

***

### deduplicationKey

> **deduplicationKey**: `string`

Defined in: [generated.ts:458](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L458)

Consumer-side idempotency key; stable across redeliveries.

***

### deliveryAttempt

> **deliveryAttempt**: `number`

Defined in: [generated.ts:460](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L460)

Delivery counter, starting at 1.

***

### eventId

> **eventId**: `string`

Defined in: [generated.ts:444](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L444)

Unique event id (outbox row identity).

***

### eventType

> **eventType**: `string`

Defined in: [generated.ts:446](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L446)

Dotted event type (e.g. `library.title.added`).

***

### invocationSequence

> **invocationSequence**: `number`

Defined in: [generated.ts:468](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L468)

Resume ordinal; distinct from `deliveryAttempt`.

***

### occurredAtUnixMs

> **occurredAtUnixMs**: `number`

Defined in: [generated.ts:450](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L450)

When the producer observed the fact.

***

### payload

> **payload**: `Uint8Array`

Defined in: [generated.ts:462](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L462)

Encoded event payload; at most `maxEventPayloadBytes`.

***

### resumePending

> **resumePending**: `boolean`

Defined in: [generated.ts:470](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L470)

True when this delivery resumes a prior suspension.

***

### schemaVersion

> **schemaVersion**: `number`

Defined in: [generated.ts:448](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L448)

Schema version of `payload`, owned by the event type.

***

### source

> **source**: `string`

Defined in: [generated.ts:472](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L472)

Append-only. Producer plugin id; empty when unknown.
