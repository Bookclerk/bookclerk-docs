[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CatalogHit

# Interface: CatalogHit

Defined in: [generated.ts:1864](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1864)

One catalog / candidate hit returned by `searchCatalog`,
`expandCandidates`, `listDeals`, and `catalogDetail`.

## Properties

### abridgement

> **abridgement**: `"unknown"` \| `"unabridged"` \| `"abridged"`

Defined in: [generated.ts:1972](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1972)

Whether the edition is abridged when the storefront says so.

***

### asin?

> `optional` **asin?**: `string`

Defined in: [generated.ts:1893](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1893)

Amazon ASIN when known.
Omitted when absent (wire zero value).

***

### authors?

> `optional` **authors?**: `string`

Defined in: [generated.ts:1873](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1873)

Authors string when known.
Omitted when absent (wire zero value).

***

### categories?

> `optional` **categories?**: `string`

Defined in: [generated.ts:1940](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1940)

Category / genre labels as a single string when known.
Omitted when absent (wire zero value).

***

### coverUrl?

> `optional` **coverUrl?**: `string`

Defined in: [generated.ts:1908](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1908)

Cover image URL.
Omitted when absent (wire zero value).

***

### currency?

> `optional` **currency?**: `string`

Defined in: [generated.ts:1955](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1955)

ISO currency code for `priceCents`.
Omitted when absent (wire zero value).

***

### description?

> `optional` **description?**: `string`

Defined in: [generated.ts:1920](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1920)

Long description / blurb when fetched.
Omitted when absent (wire zero value).

***

### isbn?

> `optional` **isbn?**: `string`

Defined in: [generated.ts:1898](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1898)

ISBN when known.
Omitted when absent (wire zero value).

***

### language?

> `optional` **language?**: `string`

Defined in: [generated.ts:1945](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1945)

Content language code when known.
Omitted when absent (wire zero value).

***

### lengthMinutes?

> `optional` **lengthMinutes?**: `bigint`

Defined in: [generated.ts:1930](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1930)

Runtime in whole minutes.
Omitted when absent (wire zero value).

***

### narrators?

> `optional` **narrators?**: `string`

Defined in: [generated.ts:1878](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1878)

Narrators string when known.
Omitted when absent (wire zero value).

***

### origin

> **origin**: `string`

Defined in: [generated.ts:1910](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1910)

Hit origin label (plugin id or storefront name).

***

### priceCents?

> `optional` **priceCents?**: `bigint`

Defined in: [generated.ts:1950](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1950)

Current price in minor units (cents).
Omitted when absent (wire zero value).

***

### priceLabel?

> `optional` **priceLabel?**: `string`

Defined in: [generated.ts:1960](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1960)

Pre-formatted price for display.
Omitted when absent (wire zero value).

***

### productId

> **productId**: `string`

Defined in: [generated.ts:1866](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1866)

Storefront product / SKU id.

***

### publishedAt?

> `optional` **publishedAt?**: `string`

Defined in: [generated.ts:1935](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1935)

Publication date string as provided by the storefront.
Omitted when absent (wire zero value).

***

### publisher?

> `optional` **publisher?**: `string`

Defined in: [generated.ts:1925](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1925)

Publisher name when known.
Omitted when absent (wire zero value).

***

### ratingCount?

> `optional` **ratingCount?**: `bigint`

Defined in: [generated.ts:1970](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1970)

Number of ratings when known.
Omitted when absent (wire zero value).

***

### ratingOverall?

> `optional` **ratingOverall?**: `number`

Defined in: [generated.ts:1965](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1965)

Aggregate rating when known.
Omitted when absent (wire zero value).

***

### series?

> `optional` **series?**: `string`

Defined in: [generated.ts:1883](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1883)

Series name when applicable.
Omitted when absent (wire zero value).

***

### seriesIndex?

> `optional` **seriesIndex?**: `string`

Defined in: [generated.ts:1888](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1888)

Series index / sequence label.
Omitted when absent (wire zero value).

***

### subtitle?

> `optional` **subtitle?**: `string`

Defined in: [generated.ts:1915](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1915)

Subtitle when distinct from `title`.
Omitted when absent (wire zero value).

***

### title

> **title**: `string`

Defined in: [generated.ts:1868](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1868)

Primary title.

***

### url?

> `optional` **url?**: `string`

Defined in: [generated.ts:1903](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1903)

Storefront product page URL.
Omitted when absent (wire zero value).
