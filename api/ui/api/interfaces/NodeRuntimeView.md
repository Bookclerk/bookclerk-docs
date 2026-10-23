[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / NodeRuntimeView

# Interface: NodeRuntimeView

Defined in: [api.ts:2139](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2139)

A node's runtime document (`node.runtime`): its label, the share of its
detected capacity the scheduler may commit, and the drain switch.

## Properties

### cpu\_budget\_percent

> **cpu\_budget\_percent**: `number`

Defined in: [api.ts:2142](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2142)

Share of detected CPU the scheduler may commit (1..=100).

***

### label

> **label**: `string`

Defined in: [api.ts:2140](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2140)

***

### memory\_budget\_percent

> **memory\_budget\_percent**: `number`

Defined in: [api.ts:2144](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2144)

Share of detected memory the scheduler may commit (1..=100).

***

### schedulable

> **schedulable**: `boolean`

Defined in: [api.ts:2146](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2146)

False drains the node: nothing new is placed there.
