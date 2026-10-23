[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PullOk

# Interface: PullOk

Defined in: [generated.ts:624](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L624)

Success payload of `ByteSource.pull`.

## Properties

### chunk

> **chunk**: `Uint8Array`

Defined in: [generated.ts:626](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L626)

Next bytes; may be shorter than requested.

***

### done

> **done**: `boolean`

Defined in: [generated.ts:628](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L628)

True when the stream is exhausted after `chunk`.
