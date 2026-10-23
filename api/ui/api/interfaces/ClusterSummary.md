[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ClusterSummary

# Interface: ClusterSummary

Defined in: [api.ts:2223](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2223)

Cluster-wide health from `GET /api/cluster/status`.

## Properties

### active\_jobs

> **active\_jobs**: `number`

Defined in: [api.ts:2233](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2233)

***

### cpu\_budget\_millicores

> **cpu\_budget\_millicores**: `number`

Defined in: [api.ts:2228](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2228)

***

### cpu\_reserved\_millicores

> **cpu\_reserved\_millicores**: `number`

Defined in: [api.ts:2229](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2229)

***

### deliveries

> **deliveries**: `object`

Defined in: [api.ts:2234](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2234)

#### dead\_letter

> **dead\_letter**: `number`

#### oldest\_pending\_age\_secs

> **oldest\_pending\_age\_secs**: `number` \| `null`

#### pending

> **pending**: `number`

#### running

> **running**: `number`

***

### drained

> **drained**: `number`

Defined in: [api.ts:2226](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2226)

***

### host\_nodes

> **host\_nodes**: `number`

Defined in: [api.ts:2227](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2227)

***

### live

> **live**: `number`

Defined in: [api.ts:2225](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2225)

***

### memory\_budget\_bytes

> **memory\_budget\_bytes**: `number`

Defined in: [api.ts:2230](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2230)

***

### memory\_reserved\_bytes

> **memory\_reserved\_bytes**: `number`

Defined in: [api.ts:2231](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2231)

***

### nodes

> **nodes**: `number`

Defined in: [api.ts:2224](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2224)

***

### running\_guests

> **running\_guests**: `number`

Defined in: [api.ts:2232](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2232)
