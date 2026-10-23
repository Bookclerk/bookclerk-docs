[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / RemoteGrantView

# Interface: RemoteGrantView

Defined in: [api.ts:2342](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2342)

The parts of a plugin grant the consent screen shows. Every field is
optional because the screen renders whatever the host sent; a missing set
means the grant asks for none of it, and a missing or `null` limit leaves
that limit to the version's manifest.

## Properties

### addressCidrs?

> `optional` **addressCidrs?**: `string`[]

Defined in: [api.ts:2352](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2352)

CIDR ranges beyond the public Internet (private or local addresses).

***

### allowUndeclaredPublicRedirects?

> `optional` **allowUndeclaredPublicRedirects?**: `boolean`

Defined in: [api.ts:2364](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2364)

Follow a redirect to a public host it did not declare.

***

### bindings?

> `optional` **bindings?**: `string`[]

Defined in: [api.ts:2354](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2354)

Host bindings (`config`, `secrets`, `kv`, …).

***

### compatibilityFlags?

> `optional` **compatibilityFlags?**: `string`[]

Defined in: [api.ts:2366](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2366)

workerd compatibility flags the isolate runs with.

***

### consumers?

> `optional` **consumers?**: `unknown`[]

Defined in: [api.ts:2362](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2362)

Events and queues it consumes (shape owned by the host).

***

### cpuMs?

> `optional` **cpuMs?**: `number` \| `null`

Defined in: [api.ts:2374](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2374)

workerd CPU budget per request, in milliseconds.

***

### cpuRatePercent?

> `optional` **cpuRatePercent?**: `number` \| `null`

Defined in: [api.ts:2370](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2370)

CPU rate as a percent of one logical CPU (above 100 asks for more than one core).

***

### diskMib?

> `optional` **diskMib?**: `number` \| `null`

Defined in: [api.ts:2372](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2372)

Disk budget in MiB for each of its `data/` and `tmp/` directories.

***

### domains?

> `optional` **domains?**: `string`[]

Defined in: [api.ts:2348](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2348)

Outbound domain patterns the guest may dial first (redirect hops aside).

***

### entrypoints?

> `optional` **entrypoints?**: `string`[]

Defined in: [api.ts:2358](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2358)

Named entrypoints it exports (`storefront`, `storage`, `cli`, …).

***

### extraProcesses?

> `optional` **extraProcesses?**: `number` \| `null`

Defined in: [api.ts:2378](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2378)

Processes or threads allowed beyond the launcher's own (native guests).

***

### jobs?

> `optional` **jobs?**: `string`[]

Defined in: [api.ts:2356](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2356)

`[triggers].jobs` types it may handle.

***

### memoryMib?

> `optional` **memoryMib?**: `number` \| `null`

Defined in: [api.ts:2368](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2368)

Memory ceiling in MiB.

***

### networkMode?

> `optional` **networkMode?**: `string`

Defined in: [api.ts:2346](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2346)

`deny` (no egress) or `outbound`; anything else is treated as `deny`.

***

### pluginId?

> `optional` **pluginId?**: `string`

Defined in: [api.ts:2344](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2344)

Display alias from `plugin.toml` (not security authoritative).

***

### producers?

> `optional` **producers?**: `string`[]

Defined in: [api.ts:2360](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2360)

Event types it may publish and queue bindings it may send to.

***

### subrequests?

> `optional` **subrequests?**: `number` \| `null`

Defined in: [api.ts:2376](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2376)

workerd outbound fetch (subrequest) budget per request.

***

### tcp?

> `optional` **tcp?**: `object`[]

Defined in: [api.ts:2350](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2350)

Raw TCP destinations: a host and the ports allowed on it.

#### host

> **host**: `string`

#### ports

> **ports**: `number`[]
