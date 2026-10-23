[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueResult

# Type Alias: QueueResult

> **QueueResult** = \{ `kind`: `"ack"`; \} \| \{ `kind`: `"retry"`; `value`: [`QueueRetry`](../interfaces/QueueRetry.md); \}

Defined in: [generated.ts:2285](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2285)

Per-message outcome inside a `QueueConsumer.queue` batch reply.
