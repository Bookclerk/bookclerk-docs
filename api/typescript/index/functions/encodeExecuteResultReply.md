[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / encodeExecuteResultReply

# Function: encodeExecuteResultReply()

> **encodeExecuteResultReply**(`outcome`): `Uint8Array`

Defined in: [db-execute.ts:359](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L359)

Encodes `ExecuteResultReply` (`ok` or `err`).

## Parameters

### outcome

\{ `ok`: [`ExecuteReply`](../interfaces/ExecuteReply.md); \} \| \{ `err`: \{ `code`: `string`; `message`: `string`; \}; \}

Successful reply or a wire `PluginError`.

## Returns

`Uint8Array`

Unpacked Cap'n stream bytes.
