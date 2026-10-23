[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / jobOutcomeFor

# Function: jobOutcomeFor()

> **jobOutcomeFor**(`job`, `returned`, `failed`): [`JobOutcomeRecord`](../type-aliases/JobOutcomeRecord.md)

Defined in: [plugin.ts:1399](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1399)

Translate a `job()` handler's return / throw into the wire outcome.

## Parameters

### job

[`JobController`](../classes/JobController.md)

Controller after the handler ran.

### returned

`void` \| [`JobCompletion`](../interfaces/JobCompletion.md) \| `undefined`

Handler return value.

### failed

`unknown`

Error thrown by the handler, if any.

## Returns

[`JobOutcomeRecord`](../type-aliases/JobOutcomeRecord.md)

Wire job outcome.
