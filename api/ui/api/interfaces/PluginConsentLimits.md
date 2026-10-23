[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginConsentLimits

# Interface: PluginConsentLimits

Defined in: [api.ts:1804](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1804)

Host-capped consent limit defaults (workerd budgets + shared jail/disk).

Jail CPU fields are cores (2 d.p.) from the daemon — no percent conversion.

## Properties

### cpu\_cores

> **cpu\_cores**: `number`

Defined in: [api.ts:1814](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1814)

Manifest / default jail CPU in cores.

***

### cpu\_ms

> **cpu\_ms**: `number`

Defined in: [api.ts:1805](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1805)

***

### disk\_mib

> **disk\_mib**: `number`

Defined in: [api.ts:1809](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1809)

***

### extra\_processes

> **extra\_processes**: `number`

Defined in: [api.ts:1820](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1820)

Default extra process/thread budget (native).

***

### jail\_cpu\_cores?

> `optional` **jail\_cpu\_cores?**: `number` \| `null`

Defined in: [api.ts:1818](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1818)

Optional Settings global jail CPU ceiling in cores.

***

### known\_bindings

> **known\_bindings**: `string`[]

Defined in: [api.ts:1823](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1823)

***

### max\_cpu\_cores

> **max\_cpu\_cores**: `number`

Defined in: [api.ts:1816](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1816)

Host max jail CPU in cores.

***

### max\_cpu\_ms

> **max\_cpu\_ms**: `number`

Defined in: [api.ts:1807](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1807)

***

### max\_disk\_mib

> **max\_disk\_mib**: `number`

Defined in: [api.ts:1810](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1810)

***

### max\_extra\_processes

> **max\_extra\_processes**: `number`

Defined in: [api.ts:1822](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1822)

Host hard cap for extra process budget.

***

### max\_memory\_mib

> **max\_memory\_mib**: `number`

Defined in: [api.ts:1812](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1812)

***

### max\_subrequests

> **max\_subrequests**: `number`

Defined in: [api.ts:1808](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1808)

***

### memory\_mib

> **memory\_mib**: `number`

Defined in: [api.ts:1811](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1811)

***

### subrequests

> **subrequests**: `number`

Defined in: [api.ts:1806](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1806)
