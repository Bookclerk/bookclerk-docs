[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / createDatabaseBinding

# Function: createDatabaseBinding()

> **createDatabaseBinding**(`transport`, `options?`): [`DatabaseBinding`](../interfaces/DatabaseBinding.md)

Defined in: [db-execute.ts:558](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L558)

Builds a host-mediated [DatabaseBinding](../interfaces/DatabaseBinding.md) over an `execute` transport.

## Parameters

### transport

[`AtomicTransport`](../interfaces/AtomicTransport.md)

Host session projection.

### options?

[`DatabaseBindingOptions`](../interfaces/DatabaseBindingOptions.md) = `{}`

Request-budget and idempotency knobs.

## Returns

[`DatabaseBinding`](../interfaces/DatabaseBinding.md)

Binding whose `execute(batch)` encodes Cap'n then calls the host.
