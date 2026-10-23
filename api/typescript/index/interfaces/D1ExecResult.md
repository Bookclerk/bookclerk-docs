[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / D1ExecResult

# Interface: D1ExecResult

Defined in: [db-execute.ts:179](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L179)

Cloudflare [D1ExecResult](https://developers.cloudflare.com/d1/worker-api/return-object/).

## Properties

### count

> **count**: `number`

Defined in: [db-execute.ts:181](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L181)

Number of statements executed (one per non-empty `exec` line).

***

### duration

> **duration**: `number`

Defined in: [db-execute.ts:183](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L183)

Total duration in milliseconds.
