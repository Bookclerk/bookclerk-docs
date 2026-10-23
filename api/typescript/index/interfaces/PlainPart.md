[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PlainPart

# Interface: PlainPart

Defined in: [generated.ts:1670](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1670)

One plain audio part written under the cache directory.

## Properties

### durationMs?

> `optional` **durationMs?**: `number`

Defined in: [generated.ts:1682](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1682)

Duration of this part in milliseconds when known.
Omitted when absent (wire zero value).

***

### path

> **path**: `string`

Defined in: [generated.ts:1672](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1672)

Absolute path to the part file under `cacheDir`.

***

### title?

> `optional` **title?**: `string`

Defined in: [generated.ts:1677](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1677)

Part title (disc / chapter label).
Omitted when absent (wire zero value).
