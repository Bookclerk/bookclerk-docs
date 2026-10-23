[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueSendMessage

# Interface: QueueSendMessage

Defined in: [generated.ts:2305](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2305)

One message to send through a `[[queues.producers]]` binding.

## Properties

### body

> **body**: `Uint8Array`

Defined in: [generated.ts:2307](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2307)

Encoded body; at most `maxEventPayloadBytes`.

***

### contentType

> **contentType**: `string`

Defined in: [generated.ts:2309](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2309)

Body encoding: `json` (default when empty), `text`, or `bytes`.

***

### delaySeconds

> **delaySeconds**: `number`

Defined in: [generated.ts:2311](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2311)

Seconds before the message may be delivered; zero means now.
