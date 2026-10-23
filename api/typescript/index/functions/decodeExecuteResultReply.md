[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / decodeExecuteResultReply

# Function: decodeExecuteResultReply()

> **decodeExecuteResultReply**(`bytes`): [`ExecuteReply`](../interfaces/ExecuteReply.md)

Defined in: [db-execute.ts:383](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L383)

Decodes a standalone `ExecuteResultReply`. `err` is thrown as `PluginError`.

## Parameters

### bytes

`Uint8Array`

Unpacked Cap'n stream.

## Returns

[`ExecuteReply`](../interfaces/ExecuteReply.md)

Structured `ExecuteReply`.

## Throws

When the union is `err` or the buffer is invalid.
