[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DbTiming

# Interface: DbTiming

Defined in: [db-execute.ts:102](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L102)

Handler/engine timing on `ExecuteReply`.

## Properties

### attemptElapsedUs

> **attemptElapsedUs**: `number`

Defined in: [db-execute.ts:104](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L104)

Whole-attempt elapsed microseconds.

***

### dbExecutionUs

> **dbExecutionUs**: `number`

Defined in: [db-execute.ts:106](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L106)

Engine-reported SQL microseconds when available.

***

### dbTimingSource

> **dbTimingSource**: `string`

Defined in: [db-execute.ts:108](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L108)

Observability source (`sqlite_txn`, `d1_sql_duration`, …).
