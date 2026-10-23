[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ExtensibleConfig

# Interface: ExtensibleConfig

Defined in: [generated.ts:230](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L230)

Plugin-specific extensible config. Not a substitute for typed ABI fields.

## Properties

### mediaType

> **mediaType**: `string`

Defined in: [generated.ts:234](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L234)

Media type of `payload` (e.g. `application/json`).

***

### payload

> **payload**: `Uint8Array`

Defined in: [generated.ts:236](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L236)

Bounded encoded payload; at most `maxConfigPayloadBytes`.

***

### schemaVersion

> **schemaVersion**: `number`

Defined in: [generated.ts:232](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L232)

Version of `payload`'s schema, owned by the plugin.
