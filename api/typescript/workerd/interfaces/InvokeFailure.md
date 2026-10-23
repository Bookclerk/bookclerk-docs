[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [workerd](../README.md) / InvokeFailure

# Interface: InvokeFailure

Defined in: [invoke.ts:91](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L91)

**`Internal`**

Transport-level `/invoke` failure as a plain value (Workers RPC drops own
properties of thrown errors, so the adapter returns this instead of
throwing across the service binding).

## Properties

### error

> **error**: `object`

Defined in: [invoke.ts:95](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L95)

JSON error body.

#### code

> **code**: `string`

#### message

> **message**: `string`

***

### status

> **status**: `number`

Defined in: [invoke.ts:93](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L93)

HTTP status the bridge answers with (`400` / `404` / `500`).
