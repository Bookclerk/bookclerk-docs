[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Brand

# Interface: Brand

Defined in: [generated.ts:1186](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1186)

Portal brand crossing the RPC boundary. Distinct from `plugin.toml`
`logo`: `iconUrl` is the live URL or data URI the SPA renders.

## Properties

### accent

> **accent**: `string`

Defined in: [generated.ts:1196](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1196)

Accent CSS color for highlights / CTAs.

***

### bg

> **bg**: `string`

Defined in: [generated.ts:1192](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1192)

Background CSS color (hex or named).

***

### fg

> **fg**: `string`

Defined in: [generated.ts:1194](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1194)

Foreground CSS color for text on `bg`.

***

### iconUrl?

> `optional` **iconUrl?**: `string`

Defined in: [generated.ts:1201](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1201)

Icon URL or data URI for the portal.
Omitted when absent (wire zero value).

***

### id

> **id**: `string`

Defined in: [generated.ts:1188](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1188)

Brand id (often matches the plugin id); empty means "no brand".

***

### name

> **name**: `string`

Defined in: [generated.ts:1190](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1190)

Display name shown next to the brand swatch.
