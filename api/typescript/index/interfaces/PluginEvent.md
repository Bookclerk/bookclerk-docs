[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginEvent

# Interface: PluginEvent

Defined in: [generated.ts:2195](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2195)

Guest-published domain event (`EventPublisher.publish`). The host stamps
`eventId`, `source` (the plugin id), and `accountId` from the invocation;
guests cannot forge either.

## Properties

### causationId

> **causationId**: `string`

Defined in: [generated.ts:2216](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2216)

Id of the event or command that caused this one; empty inherits
`Invocation.causationId`.

***

### correlationId

> **correlationId**: `string`

Defined in: [generated.ts:2211](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2211)

Trace correlation id; empty inherits `Invocation.correlationId`.

***

### deduplicationKey

> **deduplicationKey**: `string`

Defined in: [generated.ts:2205](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2205)

Producer idempotency key, unique per (account, source, eventType) in the
outbox; a repeat returns `PublishOk.duplicate = true` with the earlier id.
Empty publishes unconditionally.

***

### eventType

> **eventType**: `string`

Defined in: [generated.ts:2197](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2197)

Snake_case event type; must be listed in `[[events.producers]]`.

***

### occurredAtUnixMs

> **occurredAtUnixMs**: `number`

Defined in: [generated.ts:2209](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2209)

When the producer observed the fact; zero means "now" on the host clock.

***

### payload

> **payload**: `Uint8Array`

Defined in: [generated.ts:2207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2207)

Encoded event payload; at most `maxEventPayloadBytes`.

***

### schemaVersion

> **schemaVersion**: `number`

Defined in: [generated.ts:2199](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2199)

Schema version of `payload`, owned by the event type.
