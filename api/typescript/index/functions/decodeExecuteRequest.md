[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / decodeExecuteRequest

# Function: decodeExecuteRequest()

> **decodeExecuteRequest**(`bytes`): [`ExecuteRequest`](../interfaces/ExecuteRequest.md)

Defined in: [db-execute.ts:325](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L325)

Decodes a standalone unpacked Cap'n `ExecuteRequest` message.

## Parameters

### bytes

`Uint8Array`

Unpacked Cap'n stream.

## Returns

[`ExecuteRequest`](../interfaces/ExecuteRequest.md)

Structured request.

## Throws

When the buffer is not a valid non-empty `ExecuteRequest`.
