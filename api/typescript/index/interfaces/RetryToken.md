[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / RetryToken

# Interface: RetryToken

Defined in: [db-execute.ts:139](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L139)

Explicit retry identity. Reuses both `operationId` and `requestHash`.

## Properties

### operationId

> **operationId**: `string`

Defined in: [db-execute.ts:141](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L141)

Caller-chosen idempotency key.

***

### requestHash

> **requestHash**: `string`

Defined in: [db-execute.ts:143](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L143)

Canonical Cap'n request hash stamped by the host.
