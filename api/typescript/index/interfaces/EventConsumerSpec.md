[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventConsumerSpec

# Interface: EventConsumerSpec

Defined in: [generated.ts:1147](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1147)

One declared event consumer: a `[[events.consumers]]` row the default
entrypoint's `event(batch)` handler accepts.

## Properties

### eventType

> **eventType**: `string`

Defined in: [generated.ts:1149](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1149)

Versioned event type (snake_case, e.g. `book_acquired`).

***

### schemaVersions

> **schemaVersions**: `number`[]

Defined in: [generated.ts:1151](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1151)

Schema versions the guest can consume; never empty.

***

### supportsSuspend

> **supportsSuspend**: `boolean`

Defined in: [generated.ts:1153](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1153)

Whether `EventResult.suspended` is supported for this type.
