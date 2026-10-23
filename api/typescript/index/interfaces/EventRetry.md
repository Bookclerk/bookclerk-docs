[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventRetry

# Interface: EventRetry

Defined in: [generated.ts:482](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L482)

Redeliver later.

## Properties

### reason

> **reason**: `string`

Defined in: [generated.ts:486](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L486)

Short human reason.

***

### retryAtUnixMs

> **retryAtUnixMs**: `number`

Defined in: [generated.ts:484](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L484)

Earliest redelivery time; `0` lets the host choose.
