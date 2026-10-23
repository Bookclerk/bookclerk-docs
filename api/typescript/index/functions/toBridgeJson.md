[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / toBridgeJson

# Function: toBridgeJson()

> **toBridgeJson**(`value`): `unknown`

Defined in: [plugin.ts:2850](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2850)

Typed ABI value → bridge JSON: Cap'n `Data` fields (`Uint8Array`) travel as
base64 text; `bookclerk-workerd` decodes with the same convention.

## Parameters

### value

`unknown`

Plain typed struct value.

## Returns

`unknown`

JSON-safe projection.
