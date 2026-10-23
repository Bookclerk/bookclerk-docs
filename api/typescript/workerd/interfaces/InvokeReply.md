[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [workerd](../README.md) / InvokeReply

# Interface: InvokeReply

Defined in: [invoke.ts:77](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L77)

**`Internal`**

Successful `/invoke` reply: `$Results` bytes plus the descriptors of
capabilities the reply exported.

## Properties

### body

> **body**: `Uint8Array`

Defined in: [invoke.ts:79](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L79)

Unpacked single-segment Cap'n message (`<Interface>.<method>$Results`).

***

### caps

> **caps**: [`CapDescriptor`](CapDescriptor.md)[]

Defined in: [invoke.ts:81](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L81)

Reply capability table (`X-Bookclerk-Caps` on the response).
