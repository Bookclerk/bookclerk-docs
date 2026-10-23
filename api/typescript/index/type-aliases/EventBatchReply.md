[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventBatchReply

# Type Alias: EventBatchReply

> **EventBatchReply** = \{ `kind`: `"ok"`; `value`: [`EventResult`](EventResult.md)[]; \} \| \{ `kind`: `"err"`; `value`: `PluginError`; \}

Defined in: [generated.ts:541](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L541)

Result union of `EventConsumer.event`. `ok` has exactly one entry per
`EventBatch.events` entry, in order; a short list is a host-side error and
the missing tail is retried.
