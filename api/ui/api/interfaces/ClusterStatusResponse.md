[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ClusterStatusResponse

# Interface: ClusterStatusResponse

Defined in: [api.ts:2243](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2243)

`GET /api/cluster/status`: cluster health plus the nodes the caller may see.

## Extends

- [`NodesResponse`](NodesResponse.md)

## Properties

### capabilities

> **capabilities**: [`NodeCapabilities`](NodeCapabilities.md)

Defined in: [api.ts:2208](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2208)

#### Inherited from

[`NodesResponse`](NodesResponse.md).[`capabilities`](NodesResponse.md#capabilities)

***

### cluster

> **cluster**: [`ClusterSummary`](ClusterSummary.md)

Defined in: [api.ts:2244](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2244)

***

### node\_id

> **node\_id**: `string`

Defined in: [api.ts:2206](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2206)

#### Inherited from

[`NodesResponse`](NodesResponse.md).[`node_id`](NodesResponse.md#node_id)

***

### nodes

> **nodes**: [`ConfigNodeView`](ConfigNodeView.md)[]

Defined in: [api.ts:2207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2207)

#### Inherited from

[`NodesResponse`](NodesResponse.md).[`nodes`](NodesResponse.md#nodes)
