[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / RetryOptions

# Interface: RetryOptions

Defined in: [plugin.ts:759](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L759)

Options for [EventMessage.retry](../classes/EventMessage.md#retry).

## Properties

### delaySeconds?

> `optional` **delaySeconds?**: `number`

Defined in: [plugin.ts:763](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L763)

Delay from now, in seconds; ignored when `retryAt` is set.

***

### reason?

> `optional` **reason?**: `string`

Defined in: [plugin.ts:765](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L765)

Short operator-facing reason.

***

### retryAt?

> `optional` **retryAt?**: [`Instant`](../type-aliases/Instant.md)

Defined in: [plugin.ts:761](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L761)

Explicit earliest redelivery instant.
