[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [workerd](../README.md) / fromBridgeJson

# Function: fromBridgeJson()

> **fromBridgeJson**(`value`): `unknown`

Defined in: [plugin.ts:2823](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2823)

Bridge JSON → typed ABI value: base64 `Data` fields (`payload`, `sha256`,
`credentials`) become `Uint8Array`; everything else is structurally
identical, so no per-struct codec is needed.

## Parameters

### value

`unknown`

Bridge JSON value.

## Returns

`unknown`

Typed struct value.
