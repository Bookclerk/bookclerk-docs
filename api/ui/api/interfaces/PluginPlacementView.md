[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginPlacementView

# Interface: PluginPlacementView

Defined in: [api.ts:1648](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1648)

Scheduler placement of a plugin instance: the operator's intent and the
last scheduler pass's result.

## Properties

### constraints

> **constraints**: `string`[]

Defined in: [api.ts:1655](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1655)

Label constraints (`nas`, `zone=eu`, `!slow`, `host=<id>`).

***

### detail

> **detail**: `string`

Defined in: [api.ts:1661](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1661)

Why fewer than requested are placed; empty when satisfied.

***

### placed

> **placed**: `number`

Defined in: [api.ts:1659](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1659)

Replicas the last scheduler pass placed.

***

### replicas

> **replicas**: `number` \| `"all"`

Defined in: [api.ts:1653](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1653)

Nodes that should run the instance: a count (0 stops every replica) or
`"all"` (every live, schedulable host with budget room).

***

### revision

> **revision**: `number`

Defined in: [api.ts:1657](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1657)

Compare-and-swap revision of the intent.

***

### scheduled\_at

> **scheduled\_at**: `string`

Defined in: [api.ts:1662](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1662)

***

### updated\_at

> **updated\_at**: `string`

Defined in: [api.ts:1663](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1663)
