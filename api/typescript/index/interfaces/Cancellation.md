[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Cancellation

# Interface: Cancellation

Defined in: [generated.ts:798](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L798)

Transport cancellation. SDKs project this into a locally created AbortSignal
(AbortSignal is not a serializable Workers RPC value).

## Methods

### poll()

> **poll**(): `Promise`\<`boolean`\>

Defined in: [generated.ts:804](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L804)

Non-blocking check; `true` once the host has fenced the invocation.

#### Returns

`Promise`\<`boolean`\>

`cancelled` (boolean)
