[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginSettingsGroup

# Interface: PluginSettingsGroup

Defined in: [api.ts:1669](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1669)

Settings group for one plugin instance, or for a bootstrap database adapter.

## Properties

### config\_revision?

> `optional` **config\_revision?**: `number`

Defined in: [api.ts:1687](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1687)

Instance document revision for compare-and-swap writes.

***

### deployment?

> `optional` **deployment?**: [`PluginDeploymentView`](PluginDeploymentView.md)

Defined in: [api.ts:1689](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1689)

This node's deployment of the instance, when one exists.

***

### display\_name?

> `optional` **display\_name?**: `string`

Defined in: [api.ts:1683](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1683)

Operator-facing instance name, when set.

***

### family

> **family**: `string`

Defined in: [api.ts:1672](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1672)

Handler family (`source`, `integration`, `output`, `database`).

***

### id

> **id**: `string`

Defined in: [api.ts:1670](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1670)

***

### is\_default?

> `optional` **is\_default?**: `boolean`

Defined in: [api.ts:1685](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1685)

True when a bare plugin key or alias routes to this instance.

***

### logo?

> `optional` **logo?**: `string`

Defined in: [api.ts:1678](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1678)

Google favicon (or portal brand) URL for Settings list rows.

***

### placement?

> `optional` **placement?**: [`PluginPlacementView`](PluginPlacementView.md)

Defined in: [api.ts:1691](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1691)

Scheduler placement; absent when the instance is managed by hand.

***

### plugin\_instance\_id?

> `optional` **plugin\_instance\_id?**: `string`

Defined in: [api.ts:1681](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1681)

Instance this group edits; absent for a `config.toml`-backed database adapter.

***

### plugin\_key?

> `optional` **plugin\_key?**: `string`

Defined in: [api.ts:1674](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1674)

Provenance-qualified PluginKey when this row came from a discovered install.

***

### present\_on?

> `optional` **present\_on?**: `string`[]

Defined in: [api.ts:1693](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1693)

Nodes whose deployment of the instance is `present`.

***

### provenance?

> `optional` **provenance?**: `string`

Defined in: [api.ts:1676](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1676)

Host-evaluated provenance (`platform_bundled`, `verified_staged`, `installed`, …).

***

### settings

> **settings**: [`PluginSettingOption`](PluginSettingOption.md)[]

Defined in: [api.ts:1679](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1679)
