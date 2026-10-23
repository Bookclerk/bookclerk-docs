[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobOutcomeRecord

# Type Alias: JobOutcomeRecord

> **JobOutcomeRecord** = \{ `bytesCopied`: `number`; `kind`: `"completed"`; `message`: `string`; \} \| \{ `kind`: `"retryable"`; `message`: `string`; `retryAfterUnixMs`: `number`; \} \| \{ `kind`: `"rejected"`; `message`: `string`; \} \| \{ `kind`: `"cancelled"`; `message`: `string`; \} \| \{ `checkpoint`: [`JobCheckpoint`](../interfaces/JobCheckpoint.md); `kind`: `"suspended"`; `wakeAtUnixMs`: `number`; \}

Defined in: [plugin.ts:1189](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1189)

Recorded terminal / suspended job outcome (`JobOutcome` union, bridge JSON projection).
