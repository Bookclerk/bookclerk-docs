[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventPollReply

# Type Alias: EventPollReply

> **EventPollReply** = \{ `kind`: `"ok"`; `value`: [`EventPollResult`](../interfaces/EventPollResult.md); \} \| \{ `kind`: `"err"`; `value`: `PluginError`; \}

Defined in: [generated.ts:2117](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2117)

Result union of `RemoteLibrary.pollEvents`.
