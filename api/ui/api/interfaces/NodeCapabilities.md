[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / NodeCapabilities

# Interface: NodeCapabilities

Defined in: [api.ts:2196](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2196)

What the caller may do on the nodes it was sent.

## Properties

### can\_assign\_owner

> **can\_assign\_owner**: `boolean`

Defined in: [api.ts:2201](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2201)

***

### can\_edit\_labels

> **can\_edit\_labels**: `boolean`

Defined in: [api.ts:2200](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2200)

***

### can\_edit\_runtime

> **can\_edit\_runtime**: `boolean`

Defined in: [api.ts:2199](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2199)

***

### scope

> **scope**: `"all"` \| `"owned"`

Defined in: [api.ts:2198](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2198)

`all` for an operator; `owned` for a user who owns nodes.
