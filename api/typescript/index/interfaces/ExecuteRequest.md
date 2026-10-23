[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ExecuteRequest

# Interface: ExecuteRequest

Defined in: [db-execute.ts:58](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L58)

Typed `execute` request.

## Properties

### deadlineUnixMs

> **deadlineUnixMs**: `number`

Defined in: [db-execute.ts:66](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L66)

Guest-visible deadline (unix ms). Zero means omitted.

***

### operationId

> **operationId**: `string`

Defined in: [db-execute.ts:60](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L60)

Caller-chosen idempotency key.

***

### requestHash

> **requestHash**: `string`

Defined in: [db-execute.ts:62](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L62)

SHA-256 hex of the idempotency-relevant request; empty when omitted.

***

### statements

> **statements**: [`TypedDbStatement`](TypedDbStatement.md)[]

Defined in: [db-execute.ts:64](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L64)

Ordered statements (must be non-empty).
