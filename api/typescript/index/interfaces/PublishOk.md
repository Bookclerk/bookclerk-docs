[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PublishOk

# Interface: PublishOk

Defined in: [generated.ts:2220](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2220)

Success payload of `EventPublisher.publish`.

## Properties

### duplicate

> **duplicate**: `boolean`

Defined in: [generated.ts:2224](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2224)

True when `deduplicationKey` matched an existing outbox row.

***

### eventId

> **eventId**: `string`

Defined in: [generated.ts:2222](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2222)

Outbox event id (new or the earlier row when `duplicate`).
