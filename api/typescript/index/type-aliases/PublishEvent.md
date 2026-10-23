[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PublishEvent

# Type Alias: PublishEvent

> **PublishEvent** = `Pick`\<[`PluginEvent`](../interfaces/PluginEvent.md), `"eventType"`\> & `Partial`\<`Omit`\<[`PluginEvent`](../interfaces/PluginEvent.md), `"eventType"` \| `"payload"`\>\> & `object`

Defined in: [env.ts:17](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L17)

Event a guest publishes; the host stamps id, `source`, and `accountId`.

## Type Declaration

### payload?

> `optional` **payload?**: `Uint8Array` \| [`JsonObject`](JsonObject.md) \| `unknown`[] \| `string`

Payload bytes or a JSON-serializable value encoded as UTF-8 JSON.
