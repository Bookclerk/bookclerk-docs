[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ConfigOption

# Interface: ConfigOption

Defined in: [generated.ts:1205](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1205)

One discoverable config option group advertised for sources.

## Properties

### key

> **key**: `string`

Defined in: [generated.ts:1207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1207)

Config key under the plugin's `config.toml` table.

***

### label

> **label**: `string`

Defined in: [generated.ts:1209](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1209)

Operator-facing label for the option group.

***

### values

> **values**: [`ConfigOptionValue`](ConfigOptionValue.md)[]

Defined in: [generated.ts:1211](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1211)

Allowed selectable values for this key.
