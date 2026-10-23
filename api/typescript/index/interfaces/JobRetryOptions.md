[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobRetryOptions

# Interface: JobRetryOptions

Defined in: [plugin.ts:1215](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1215)

Options for [JobController.retryLater](../classes/JobController.md#retrylater).

## Properties

### reason?

> `optional` **reason?**: `string`

Defined in: [plugin.ts:1219](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1219)

Short operator-facing reason.

***

### retryAt?

> `optional` **retryAt?**: [`Instant`](../type-aliases/Instant.md)

Defined in: [plugin.ts:1217](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1217)

Earliest retry instant; omitted lets the host choose.
