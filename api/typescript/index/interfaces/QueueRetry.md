[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueRetry

# Interface: QueueRetry

Defined in: [generated.ts:2277](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2277)

Redeliver a queue message later.

## Properties

### delaySeconds

> **delaySeconds**: `number`

Defined in: [generated.ts:2279](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2279)

Seconds before redelivery; zero applies the consumer's `retry_delay`.

***

### reason

> **reason**: `string`

Defined in: [generated.ts:2281](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2281)

Short human reason.
