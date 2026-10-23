[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginDeploymentView

# Interface: PluginDeploymentView

Defined in: [api.ts:1627](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1627)

This node's deployment of a plugin instance, with its latest observation.

## Properties

### applied\_config\_revision?

> `optional` **applied\_config\_revision?**: `number`

Defined in: [api.ts:1640](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1640)

***

### deployment\_id

> **deployment\_id**: `string`

Defined in: [api.ts:1628](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1628)

***

### desired

> **desired**: `string`

Defined in: [api.ts:1635](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1635)

`present` / `absent` when an operator manages the instance (the Enabled
switch), `placed` / `unplaced` when the cluster scheduler does. The
host runs it for `present` and `placed`.

***

### detail?

> `optional` **detail?**: `string`

Defined in: [api.ts:1639](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1639)

***

### node\_id

> **node\_id**: `string`

Defined in: [api.ts:1629](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1629)

***

### observed\_at?

> `optional` **observed\_at?**: `string`

Defined in: [api.ts:1641](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1641)

***

### revision

> **revision**: `number`

Defined in: [api.ts:1636](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1636)

***

### status?

> `optional` **status?**: `string`

Defined in: [api.ts:1638](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1638)

`materialized`, `running`, `healthy`, `error`, or `retired` when observed.
