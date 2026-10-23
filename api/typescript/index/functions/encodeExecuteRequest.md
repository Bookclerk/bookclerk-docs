[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / encodeExecuteRequest

# Function: encodeExecuteRequest()

> **encodeExecuteRequest**(`request`): `Uint8Array`

Defined in: [db-execute.ts:302](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L302)

Encodes a standalone unpacked Cap'n `ExecuteRequest` message.

## Parameters

### request

[`ExecuteRequest`](../interfaces/ExecuteRequest.md)

Structured request (non-empty `statements`).

## Returns

`Uint8Array`

Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

## Throws

When `statements` is empty.
