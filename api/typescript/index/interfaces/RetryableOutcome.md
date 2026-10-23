[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / RetryableOutcome

# Interface: RetryableOutcome

Defined in: [generated.ts:404](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L404)

Job failed transiently; the host reschedules it.

## Properties

### message

> **message**: `string`

Defined in: [generated.ts:406](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L406)

Short human reason.

***

### retryAfterUnixMs

> **retryAfterUnixMs**: `number`

Defined in: [generated.ts:408](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L408)

Earliest retry time; `0` lets the host choose.
