[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / decodeExtensibleConfig

# Function: decodeExtensibleConfig()

> **decodeExtensibleConfig**(`cfg`): [`JsonObject`](../type-aliases/JsonObject.md)

Defined in: [plugin.ts:602](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L602)

Decode an `ExtensibleConfig` (`{ schemaVersion, mediaType, payload }`) into
the plain object authors read from `env.CONFIG` / `env.SECRETS`. Non-JSON
media types are surfaced verbatim so nothing is lost.

## Parameters

### cfg

`unknown`

Wire config struct, a pre-decoded object, or `undefined`.

## Returns

[`JsonObject`](../type-aliases/JsonObject.md)

Plain JSON object (`{}` when empty or malformed).
