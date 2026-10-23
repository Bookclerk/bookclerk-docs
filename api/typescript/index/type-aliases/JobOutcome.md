[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobOutcome

# Type Alias: JobOutcome

> **JobOutcome** = \{ `kind`: `"completed"`; `value`: [`CompletedOutcome`](../interfaces/CompletedOutcome.md); \} \| \{ `kind`: `"retryable"`; `value`: [`RetryableOutcome`](../interfaces/RetryableOutcome.md); \} \| \{ `kind`: `"rejected"`; `value`: [`RejectedOutcome`](../interfaces/RejectedOutcome.md); \} \| \{ `kind`: `"cancelled"`; `value`: [`CancelledOutcome`](../interfaces/CancelledOutcome.md); \} \| \{ `kind`: `"suspended"`; `value`: [`SuspendedOutcome`](../interfaces/SuspendedOutcome.md); \}

Defined in: [generated.ts:434](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L434)

Terminal or suspended result of `JobRunner.job`.
