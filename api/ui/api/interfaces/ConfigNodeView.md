[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ConfigNodeView

# Interface: ConfigNodeView

Defined in: [api.ts:2174](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2174)

One cluster node as `GET /api/config/nodes` returns it.

## Properties

### implicit\_labels

> **implicit\_labels**: `string`[]

Defined in: [api.ts:2180](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2180)

Labels every node carries (`node=<id>`, `target=<target>`).

***

### kind

> **kind**: `"owner"` \| `"host"`

Defined in: [api.ts:2190](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2190)

`owner` (shares the database) or `host` (its own database, enrolled to contribute).

***

### labels

> **labels**: `string`[]

Defined in: [api.ts:2178](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2178)

Operator labels.

***

### live

> **live**: `boolean`

Defined in: [api.ts:2182](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2182)

True while the node's daemon heartbeats.

***

### node\_id

> **node\_id**: `string`

Defined in: [api.ts:2175](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2175)

***

### owner

> **owner**: \{ `display_name`: `string` \| `null`; `role`: `string`; `user_id`: `number`; \} \| `null`

Defined in: [api.ts:2192](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2192)

User who administers this node, when one is assigned.

***

### runtime

> **runtime**: [`NodeRuntimeView`](NodeRuntimeView.md)

Defined in: [api.ts:2183](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2183)

***

### software\_version?

> `optional` **software\_version?**: `string`

Defined in: [api.ts:2186](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2186)

***

### status

> **status**: [`NodeStatusView`](NodeStatusView.md) \| `null`

Defined in: [api.ts:2185](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2185)

Latest heartbeat; null when the node's daemon never ran.

***

### target

> **target**: `string`

Defined in: [api.ts:2176](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2176)

***

### this\_node

> **this\_node**: `boolean`

Defined in: [api.ts:2188](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2188)

True for the node serving this request.
