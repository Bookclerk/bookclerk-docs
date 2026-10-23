[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PlainFetch

# Interface: PlainFetch

Defined in: [generated.ts:1697](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1697)

Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
guests decrypt before responding.

## Properties

### chapters

> **chapters**: [`ChapterMarker`](ChapterMarker.md)[]

Defined in: [generated.ts:1711](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1711)

Chapter markers; empty when unknown.

***

### coverPath?

> `optional` **coverPath?**: `string`

Defined in: [generated.ts:1709](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1709)

Cover image path under the cache directory.
Omitted when absent (wire zero value).

***

### m4bPath?

> `optional` **m4bPath?**: `string`

Defined in: [generated.ts:1704](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1704)

Single M4B path when the guest assembled one.
Omitted when absent (wire zero value).

***

### parts

> **parts**: [`PlainPart`](PlainPart.md)[]

Defined in: [generated.ts:1699](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1699)

Ordered audio part files written under the cache directory.

***

### pdfUrl?

> `optional` **pdfUrl?**: `string`

Defined in: [generated.ts:1716](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1716)

Companion PDF download URL when the store exposes one.
Omitted when absent (wire zero value).
