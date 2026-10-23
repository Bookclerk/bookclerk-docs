[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobSuspendOptions

# Interface: JobSuspendOptions

Defined in: [plugin.ts:1205](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1205)

Options for [JobController.suspend](../classes/JobController.md#suspend).

## Properties

### checkpoint?

> `optional` **checkpoint?**: [`Checkpoint`](../type-aliases/Checkpoint.md)

Defined in: [plugin.ts:1207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1207)

Bounded checkpoint replayed on resume (at most `maxCheckpointBytes`).

***

### checkpointSchemaVersion?

> `optional` **checkpointSchemaVersion?**: `number`

Defined in: [plugin.ts:1209](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1209)

Schema version of `checkpoint` (default `1`).

***

### wakeAt?

> `optional` **wakeAt?**: [`Instant`](../type-aliases/Instant.md)

Defined in: [plugin.ts:1211](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1211)

Earliest resume instant.
