[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / SuspendedOutcome

# Interface: SuspendedOutcome

Defined in: [generated.ts:424](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L424)

Job released the process and asks to be resumed later.

## Properties

### checkpointJson

> **checkpointJson**: `string`

Defined in: [generated.ts:426](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L426)

Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.

***

### checkpointSchemaVersion

> **checkpointSchemaVersion**: `number`

Defined in: [generated.ts:428](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L428)

Schema version of `checkpointJson`.

***

### wakeAtUnixMs

> **wakeAtUnixMs**: `number`

Defined in: [generated.ts:430](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L430)

Earliest resume time.
