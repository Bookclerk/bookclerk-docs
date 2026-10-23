[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchClusterStatus

# Function: fetchClusterStatus()

> **fetchClusterStatus**(): `Promise`\<[`ClusterStatusResponse`](../interfaces/ClusterStatusResponse.md)\>

Defined in: [api.ts:2253](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2253)

Cluster health every node administrator may see, with detail for the
caller's visible nodes.

## Returns

`Promise`\<[`ClusterStatusResponse`](../interfaces/ClusterStatusResponse.md)\>

Cluster summary, visible nodes, and the caller's capabilities.
