[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventPublisher

# Interface: EventPublisher

Defined in: [generated.ts:837](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L837)

`EVENTS` binding: host-served outbox publisher granted through
`Bindings.events`. Event types must be listed in `[[events.producers]]`.

## Methods

### publish()

> **publish**(`event`): `Promise`\<[`PublishReply`](../type-aliases/PublishReply.md)\>

Defined in: [generated.ts:844](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L844)

Append one event to the outbox (idempotent on `deduplicationKey`).

#### Parameters

##### event

[`PluginEvent`](PluginEvent.md)

Event to publish.

#### Returns

`Promise`\<[`PublishReply`](../type-aliases/PublishReply.md)\>

[PublishReply](../type-aliases/PublishReply.md)
