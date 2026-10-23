[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginSettingOption

# Interface: PluginSettingOption

Defined in: [api.ts:1612](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1612)

Editable plugin setting descriptor from the settings API.

For a plugin instance group the key is the bare `[[settings]]` key from the
plugin's manifest and `value` is the effective value (document over schema
default). A `secret` option never carries its value.

## Properties

### choices?

> `optional` **choices?**: [`PluginSettingChoice`](PluginSettingChoice.md)[]

Defined in: [api.ts:1617](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1617)

***

### key

> **key**: `string`

Defined in: [api.ts:1613](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1613)

***

### label

> **label**: `string`

Defined in: [api.ts:1614](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1614)

***

### secret?

> `optional` **secret?**: `boolean`

Defined in: [api.ts:1619](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1619)

True for a `secret` setting; `value` is always empty.

***

### secret\_set?

> `optional` **secret\_set?**: `boolean`

Defined in: [api.ts:1621](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1621)

For a secret setting, whether a sealed value is stored for the instance.

***

### value

> **value**: `string`

Defined in: [api.ts:1615](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1615)

***

### value\_type

> **value\_type**: `"string"` \| `"number"` \| `"boolean"` \| `"secret"`

Defined in: [api.ts:1616](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1616)
