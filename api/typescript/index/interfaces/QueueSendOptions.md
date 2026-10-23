[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueSendOptions

# Interface: QueueSendOptions

Defined in: [env.ts:38](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L38)

Per-send options for a [QueueBinding](QueueBinding.md).

## Extended by

- [`QueueSendBatchEntry`](QueueSendBatchEntry.md)

## Properties

### contentType?

> `optional` **contentType?**: `"text"` \| `"bytes"` \| `"json"`

Defined in: [env.ts:43](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L43)

Body encoding: `json` (default; the body is JSON-encoded), `text`
(UTF-8 string), or `bytes` (raw `Uint8Array`).

***

### delaySeconds?

> `optional` **delaySeconds?**: `number`

Defined in: [env.ts:45](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L45)

Seconds before the message may be delivered; `0` means now.
