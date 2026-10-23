[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / putNodeRuntime

# Function: putNodeRuntime()

> **putNodeRuntime**(`nodeId`, `body`): `Promise`\<\{ `node_id`: `string`; `revision`: `number`; `runtime`: [`NodeRuntimeView`](../interfaces/NodeRuntimeView.md); \}\>

Defined in: [api.ts:2286](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2286)

Edits a node's label, budgets, or drain switch. The node applies it at
its next heartbeat.

## Parameters

### nodeId

`string`

Node to edit.

### body

`Partial`\<[`NodeRuntimeView`](../interfaces/NodeRuntimeView.md)\>

Fields to change; omitted fields keep their value.

## Returns

`Promise`\<\{ `node_id`: `string`; `revision`: `number`; `runtime`: [`NodeRuntimeView`](../interfaces/NodeRuntimeView.md); \}\>

The stored runtime document.
