[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventPublisherBinding

# Interface: EventPublisherBinding

Defined in: [env.ts:27](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L27)

`EVENTS` binding: outbox publisher, present when `[[events.producers]]` is
declared and granted.

## Methods

### publish()

> **publish**(`event`): `Promise`\<[`PublishOk`](PublishOk.md)\>

Defined in: [env.ts:34](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L34)

Publish one domain event through the host outbox.

#### Parameters

##### event

[`PublishEvent`](../type-aliases/PublishEvent.md)

Event type, schema version, payload, dedup key.

#### Returns

`Promise`\<[`PublishOk`](PublishOk.md)\>

Outbox row identity (`duplicate` when the dedup key matched).
