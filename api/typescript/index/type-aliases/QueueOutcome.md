[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueOutcome

# Type Alias: QueueOutcome

> **QueueOutcome** = \{ `kind`: `"ack"`; \} \| \{ `delaySeconds`: `number`; `kind`: `"retry"`; `reason`: `string`; \}

Defined in: [plugin.ts:1027](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1027)

Recorded per-message outcome of a `queue()` handler (bridge projection of `QueueResult`).
