[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / jsonPayload

# Function: jsonPayload()

> **jsonPayload**(`value`): [`ExtensibleConfig`](../interfaces/ExtensibleConfig.md)

Defined in: [plugin.ts:638](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L638)

Wraps a JSON-serializable value as an `application/json` `ExtensibleConfig`
(schema version 1) — the shape of `CliInvokeResult.payload` and every other
extensible payload on the wire.

## Parameters

### value

`unknown`

JSON-serializable value.

## Returns

[`ExtensibleConfig`](../interfaces/ExtensibleConfig.md)

Wire-shaped extensible payload.
