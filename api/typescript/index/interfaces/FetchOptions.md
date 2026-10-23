[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / FetchOptions

# Interface: FetchOptions

Defined in: [generated.ts:1608](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1608)

Fetch-relevant acquire knobs the host forwards so external load matches
in-process. Packaging / naming knobs stay host-side.

## Properties

### chapterLayout

> **chapterLayout**: `string`

Defined in: [generated.ts:1631](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1631)

Preferred chapter API layout when fetching (`tree` or `flat`).

***

### coverSize

> **coverSize**: `string`

Defined in: [generated.ts:1629](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1629)

Cover image size request (`500`, `1215`, or `native`).

***

### downloadClipsBookmarks

> **downloadClipsBookmarks**: `boolean`

Defined in: [generated.ts:1635](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1635)

When true, download clips/bookmarks sidecars when offered.

***

### downloadCover

> **downloadCover**: `boolean`

Defined in: [generated.ts:1625](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1625)

When true, download a cover image alongside audio.

***

### downloadPdf

> **downloadPdf**: `boolean`

Defined in: [generated.ts:1627](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1627)

When true, download a companion PDF when the store exposes one.

***

### downloadSpeedLimitKbps

> **downloadSpeedLimitKbps**: `number`

Defined in: [generated.ts:1639](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1639)

Fetch speed cap in KB/s (`0` = unlimited).

***

### retainAaxFile

> **retainAaxFile**: `boolean`

Defined in: [generated.ts:1637](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1637)

When true, keep the encrypted download in storage.

***

### saveMetadataJson

> **saveMetadataJson**: `boolean`

Defined in: [generated.ts:1641](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1641)

When true, persist raw catalog API JSON as `metadata.json`.

***

### stripAudibleBrandAudio

> **stripAudibleBrandAudio**: `boolean`

Defined in: [generated.ts:1633](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1633)

When true, trim Audible brand intro/outro from the remux window.

***

### widevine

> **widevine**: `boolean`

Defined in: [generated.ts:1610](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1610)

Prefer Widevine/CENC download when the store offers it.

***

### widevineCdmPath?

> `optional` **widevineCdmPath?**: `string`

Defined in: [generated.ts:1617](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1617)

Local Widevine `.wvd` path granted to the guest.
Omitted when absent (wire zero value).

***

### widevineCdmProvider?

> `optional` **widevineCdmProvider?**: `string`

Defined in: [generated.ts:1623](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1623)

Remote L3 CDM provider URL; empty means the classic default, `off`
disables remote provisioning.
Omitted when absent (wire zero value).

***

### xheAac

> **xheAac**: `boolean`

Defined in: [generated.ts:1612](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1612)

Prefer xHE-AAC on the Widevine path when offered.
