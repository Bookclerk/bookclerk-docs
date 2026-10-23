[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventSuspended

# Interface: EventSuspended

Defined in: [generated.ts:506](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L506)

Append-only. Mirrors job SuspendedOutcome; event handlers persist a
bounded checkpoint and release the process until wakeAtUnixMs. Optional
wake-on-matching-event fields (empty = timestamp-only).

## Properties

### checkpointJson

> **checkpointJson**: `string`

Defined in: [generated.ts:508](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L508)

Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.

***

### checkpointSchemaVersion

> **checkpointSchemaVersion**: `number`

Defined in: [generated.ts:510](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L510)

Schema version of `checkpointJson`.

***

### wakeAtUnixMs

> **wakeAtUnixMs**: `number`

Defined in: [generated.ts:512](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L512)

Earliest resume time.

***

### wakeOnEventType

> **wakeOnEventType**: `string`

Defined in: [generated.ts:514](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L514)

Also wake when an event of this type arrives; empty disables.

***

### wakeOnFilterJson

> **wakeOnFilterJson**: `string`

Defined in: [generated.ts:516](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L516)

JSON filter applied to matching wake events; empty matches all.
