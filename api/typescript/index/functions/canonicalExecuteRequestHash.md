[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / canonicalExecuteRequestHash

# Function: canonicalExecuteRequestHash()

> **canonicalExecuteRequestHash**(`request`): `Promise`\<`string`\>

Defined in: [db-execute.ts:729](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L729)

SHA-256 hex of the Cap'n request with id and hash cleared.

## Parameters

### request

[`ExecuteRequest`](../interfaces/ExecuteRequest.md)

Structured request.

## Returns

`Promise`\<`string`\>

Hex digest matching the host canonical hash.
