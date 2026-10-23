[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / QueueSendBatchEntry

# Interface: QueueSendBatchEntry

Defined in: [env.ts:49](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L49)

One entry of [QueueBinding.sendBatch](QueueBinding.md#sendbatch).

## Extends

- [`QueueSendOptions`](QueueSendOptions.md)

## Properties

### body

> **body**: `unknown`

Defined in: [env.ts:51](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L51)

Message body (JSON value, string, or bytes per `contentType`).

***

### contentType?

> `optional` **contentType?**: `"text"` \| `"bytes"` \| `"json"`

Defined in: [env.ts:43](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L43)

Body encoding: `json` (default; the body is JSON-encoded), `text`
(UTF-8 string), or `bytes` (raw `Uint8Array`).

#### Inherited from

[`QueueSendOptions`](QueueSendOptions.md).[`contentType`](QueueSendOptions.md#contenttype)

***

### delaySeconds?

> `optional` **delaySeconds?**: `number`

Defined in: [env.ts:45](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L45)

Seconds before the message may be delivered; `0` means now.

#### Inherited from

[`QueueSendOptions`](QueueSendOptions.md).[`delaySeconds`](QueueSendOptions.md#delayseconds)
