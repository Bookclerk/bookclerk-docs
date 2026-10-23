[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PurchaseHint

# Interface: PurchaseHint

Defined in: [generated.ts:2000](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2000)

Purchase hint for SPA / CLI purchase deep-links.

## Properties

### currency?

> `optional` **currency?**: `string`

Defined in: [generated.ts:2022](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2022)

ISO currency code for price fields.
Omitted when absent (wire zero value).

***

### listPriceCents?

> `optional` **listPriceCents?**: `bigint`

Defined in: [generated.ts:2032](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2032)

List / MSRP price in minor units.
Omitted when absent (wire zero value).

***

### listPriceLabel?

> `optional` **listPriceLabel?**: `string`

Defined in: [generated.ts:2037](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2037)

Pre-formatted list price.
Omitted when absent (wire zero value).

***

### memberPriceCents?

> `optional` **memberPriceCents?**: `bigint`

Defined in: [generated.ts:2042](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2042)

Member / Plus price in minor units.
Omitted when absent (wire zero value).

***

### memberPriceLabel?

> `optional` **memberPriceLabel?**: `string`

Defined in: [generated.ts:2047](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2047)

Pre-formatted member price.
Omitted when absent (wire zero value).

***

### priceCents?

> `optional` **priceCents?**: `bigint`

Defined in: [generated.ts:2017](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2017)

Current price in minor units.
Omitted when absent (wire zero value).

***

### priceLabel?

> `optional` **priceLabel?**: `string`

Defined in: [generated.ts:2027](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2027)

Pre-formatted current price.
Omitted when absent (wire zero value).

***

### productId

> **productId**: `string`

Defined in: [generated.ts:2002](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2002)

Storefront product id.

***

### title?

> `optional` **title?**: `string`

Defined in: [generated.ts:2007](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2007)

Title when resolved.
Omitted when absent (wire zero value).

***

### url?

> `optional` **url?**: `string`

Defined in: [generated.ts:2012](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2012)

Purchase or product-page URL.
Omitted when absent (wire zero value).
