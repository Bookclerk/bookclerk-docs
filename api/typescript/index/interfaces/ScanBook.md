[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ScanBook

# Interface: ScanBook

Defined in: [generated.ts:1518](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1518)

One library title returned by `ContentSource.scan`. The host upserts these
rows and forces `source` to the plugin id.

## Properties

### accountId

> **accountId**: `string`

Defined in: [generated.ts:1520](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1520)

Account that owns this library entry.

***

### asin?

> `optional` **asin?**: `string`

Defined in: [generated.ts:1534](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1534)

Amazon ASIN when the storefront exposes one.
Omitted when absent (wire zero value).

***

### authors?

> `optional` **authors?**: `string`

Defined in: [generated.ts:1544](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1544)

Comma- or guest-formatted author list.
Omitted when absent (wire zero value).

***

### contentKind?

> `optional` **contentKind?**: `string`

Defined in: [generated.ts:1564](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1564)

Content classification (e.g. `book` vs `episode`).
Omitted when absent (wire zero value).

***

### isbn?

> `optional` **isbn?**: `string`

Defined in: [generated.ts:1539](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1539)

ISBN when the storefront exposes one.
Omitted when absent (wire zero value).

***

### lengthMinutes?

> `optional` **lengthMinutes?**: `bigint`

Defined in: [generated.ts:1574](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1574)

Runtime in whole minutes when known.
Omitted when absent (wire zero value).

***

### marketplace?

> `optional` **marketplace?**: `string`

Defined in: [generated.ts:1529](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1529)

Marketplace / region when known.
Omitted when absent (wire zero value).

***

### narrators?

> `optional` **narrators?**: `string`

Defined in: [generated.ts:1549](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1549)

Comma- or guest-formatted narrator list.
Omitted when absent (wire zero value).

***

### productId

> **productId**: `string`

Defined in: [generated.ts:1522](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1522)

Storefront product / SKU id.

***

### publisher?

> `optional` **publisher?**: `string`

Defined in: [generated.ts:1569](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1569)

Publisher name when known.
Omitted when absent (wire zero value).

***

### series?

> `optional` **series?**: `string`

Defined in: [generated.ts:1554](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1554)

Series name when applicable.
Omitted when absent (wire zero value).

***

### seriesIndex?

> `optional` **seriesIndex?**: `string`

Defined in: [generated.ts:1559](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1559)

Series index / sequence label.
Omitted when absent (wire zero value).

***

### subtitle?

> `optional` **subtitle?**: `string`

Defined in: [generated.ts:1579](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1579)

Subtitle when distinct from `title`.
Omitted when absent (wire zero value).

***

### title

> **title**: `string`

Defined in: [generated.ts:1524](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1524)

Primary title string.
