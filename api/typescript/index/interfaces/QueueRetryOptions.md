[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueRetryOptions

# Interface: QueueRetryOptions

Defined in: [plugin.ts:1019](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1019)

Options for [QueueMessage.retry](../classes/QueueMessage.md#retry).

## Properties

### delaySeconds?

> `optional` **delaySeconds?**: `number`

Defined in: [plugin.ts:1021](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1021)

Seconds before redelivery; `0` applies the consumer's `retry_delay`.

***

### reason?

> `optional` **reason?**: `string`

Defined in: [plugin.ts:1023](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1023)

Short operator-facing reason.
