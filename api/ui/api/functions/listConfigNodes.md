[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / listConfigNodes

# Function: listConfigNodes()

> **listConfigNodes**(): `Promise`\<[`NodesResponse`](../interfaces/NodesResponse.md)\>

Defined in: [api.ts:2217](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2217)

Lists the nodes the caller may see: every node for an operator, the
owned ones for a node owner.

## Returns

`Promise`\<[`NodesResponse`](../interfaces/NodesResponse.md)\>

The serving node's id, the visible nodes, and what the caller may edit.
