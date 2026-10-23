[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / NodeStatusView

# Interface: NodeStatusView

Defined in: [api.ts:2150](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2150)

A node's latest heartbeat (every 15 s while its daemon runs).

## Properties

### availability\_ppm

> **availability\_ppm**: `number`

Defined in: [api.ts:2170](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2170)

Recent online fraction in parts per million (decays over about a day).

***

### cpu\_budget\_millicores

> **cpu\_budget\_millicores**: `number`

Defined in: [api.ts:2158](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2158)

***

### cpu\_millicores

> **cpu\_millicores**: `number`

Defined in: [api.ts:2155](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2155)

Detected CPU capacity in millicores.

***

### cpu\_used\_millicores

> **cpu\_used\_millicores**: `number` \| `null`

Defined in: [api.ts:2162](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2162)

Measured CPU in use since the previous heartbeat.

***

### files\_dir\_free\_bytes

> **files\_dir\_free\_bytes**: `number` \| `null`

Defined in: [api.ts:2165](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2165)

***

### heartbeat\_at

> **heartbeat\_at**: `string`

Defined in: [api.ts:2151](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2151)

***

### interval\_ms

> **interval\_ms**: `number`

Defined in: [api.ts:2153](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2153)

***

### memory\_available\_bytes

> **memory\_available\_bytes**: `number` \| `null`

Defined in: [api.ts:2164](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2164)

Measured memory available to new work.

***

### memory\_budget\_bytes

> **memory\_budget\_bytes**: `number` \| `null`

Defined in: [api.ts:2159](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2159)

***

### memory\_bytes

> **memory\_bytes**: `number` \| `null`

Defined in: [api.ts:2157](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2157)

Detected memory in bytes, when the node could read it.

***

### reserved

> **reserved**: `object`

Defined in: [api.ts:2167](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2167)

Sum of the running guests' jail ceilings.

#### cpu\_millicores

> **cpu\_millicores**: `number`

#### memory\_bytes

> **memory\_bytes**: `number`

***

### running\_guests

> **running\_guests**: `number`

Defined in: [api.ts:2168](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2168)

***

### schedulable

> **schedulable**: `boolean`

Defined in: [api.ts:2160](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2160)

***

### started\_at

> **started\_at**: `string`

Defined in: [api.ts:2152](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2152)
