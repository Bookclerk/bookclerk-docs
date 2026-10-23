[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobCompletion

# Interface: JobCompletion

Defined in: [plugin.ts:1197](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1197)

Value a `job()` handler may return to annotate a `completed` outcome.

## Properties

### bytesCopied?

> `optional` **bytesCopied?**: `number`

Defined in: [plugin.ts:1201](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1201)

Bytes copied by the job, when meaningful.

***

### message?

> `optional` **message?**: `string`

Defined in: [plugin.ts:1199](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1199)

Operator-facing summary.
