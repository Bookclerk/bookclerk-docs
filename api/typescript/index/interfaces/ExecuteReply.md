[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ExecuteReply

# Interface: ExecuteReply

Defined in: [db-execute.ts:114](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L114)

Typed `execute` reply.

## Properties

### operationId

> **operationId**: `string`

Defined in: [db-execute.ts:116](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L116)

Echo of the request `operationId`.

***

### statements

> **statements**: [`StatementResult`](StatementResult.md)[]

Defined in: [db-execute.ts:118](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L118)

Per-statement results, in plan order.

***

### timing

> **timing**: [`DbTiming`](DbTiming.md)

Defined in: [db-execute.ts:120](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L120)

Handler/engine timing.
