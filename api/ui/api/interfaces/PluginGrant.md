[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginGrant

# Interface: PluginGrant

Defined in: [api.ts:1746](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1746)

Plugin consent grant shape serialized by the daemon.

## Properties

### addressCidrs?

> `optional` **addressCidrs?**: `string`[]

Defined in: [api.ts:1770](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1770)

***

### allowUndeclaredPublicRedirects?

> `optional` **allowUndeclaredPublicRedirects?**: `boolean`

Defined in: [api.ts:1773](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1773)

***

### approvedAt

> **approvedAt**: `string`

Defined in: [api.ts:1777](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1777)

***

### authorityRevision?

> `optional` **authorityRevision?**: `string`

Defined in: [api.ts:1774](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1774)

***

### bindings

> **bindings**: `string`[]

Defined in: [api.ts:1775](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1775)

***

### compatibilityFlags

> **compatibilityFlags**: `string`[]

Defined in: [api.ts:1776](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1776)

***

### consumers?

> `optional` **consumers?**: `object`[]

Defined in: [api.ts:1754](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1754)

Approved event-consumer subscriptions (type + schemas + suspend + filter).

#### eventType

> **eventType**: `string`

#### filter?

> `optional` **filter?**: `string` \| `null`

#### schemaVersions

> **schemaVersions**: `number`[]

#### supportsSuspend

> **supportsSuspend**: `boolean`

***

### cpuCores?

> `optional` **cpuCores?**: `number`

Defined in: [api.ts:1783](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1783)

Jail CPU in cores (2 d.p.) from the daemon.

***

### cpuMs?

> `optional` **cpuMs?**: `number`

Defined in: [api.ts:1778](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1778)

***

### diskMib?

> `optional` **diskMib?**: `number`

Defined in: [api.ts:1780](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1780)

***

### domains

> **domains**: `string`[]

Defined in: [api.ts:1763](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1763)

***

### entrypoints

> **entrypoints**: `string`[]

Defined in: [api.ts:1750](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1750)

Exported entrypoints the operator approved (`storefront`, `cli`, …).

***

### extraProcesses?

> `optional` **extraProcesses?**: `number`

Defined in: [api.ts:1785](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1785)

Extra process/thread budget beyond launcher overhead (native).

***

### jobs?

> `optional` **jobs?**: `string`[]

Defined in: [api.ts:1761](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1761)

Approved `[triggers].jobs` types.

***

### manifestDomains?

> `optional` **manifestDomains?**: `string`[]

Defined in: [api.ts:1764](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1764)

***

### memoryMib?

> `optional` **memoryMib?**: `number`

Defined in: [api.ts:1781](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1781)

***

### networkMode

> **networkMode**: `string`

Defined in: [api.ts:1762](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1762)

***

### operatorAddedCidrs?

> `optional` **operatorAddedCidrs?**: `string`[]

Defined in: [api.ts:1771](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1771)

***

### operatorAddedDomains?

> `optional` **operatorAddedDomains?**: `string`[]

Defined in: [api.ts:1765](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1765)

***

### operatorAddedTcp?

> `optional` **operatorAddedTcp?**: `object`[]

Defined in: [api.ts:1768](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1768)

#### host

> **host**: `string`

#### ports

> **ports**: `number`[]

***

### operatorDeniedCidrs?

> `optional` **operatorDeniedCidrs?**: `string`[]

Defined in: [api.ts:1772](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1772)

***

### operatorDeniedDomains?

> `optional` **operatorDeniedDomains?**: `string`[]

Defined in: [api.ts:1766](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1766)

***

### operatorDeniedTcp?

> `optional` **operatorDeniedTcp?**: `object`[]

Defined in: [api.ts:1769](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1769)

#### host

> **host**: `string`

#### ports

> **ports**: `number`[]

***

### pluginId

> **pluginId**: `string`

Defined in: [api.ts:1748](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1748)

***

### pluginKey?

> `optional` **pluginKey?**: `string`

Defined in: [api.ts:1747](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1747)

***

### producers

> **producers**: `string`[]

Defined in: [api.ts:1752](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1752)

Event types the operator approved the plugin to publish.

***

### subrequests?

> `optional` **subrequests?**: `number`

Defined in: [api.ts:1779](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1779)

***

### tcp?

> `optional` **tcp?**: `object`[]

Defined in: [api.ts:1767](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1767)

#### host

> **host**: `string`

#### ports

> **ports**: `number`[]
