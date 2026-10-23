[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueSendReply

# Type Alias: QueueSendReply

> **QueueSendReply** = \{ `kind`: `"ok"`; `value`: [`QueueSendOk`](../interfaces/QueueSendOk.md); \} \| \{ `kind`: `"err"`; `value`: `PluginError`; \}

Defined in: [generated.ts:2321](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2321)

Result union of `QueueProducer.send` / `sendBatch`.
