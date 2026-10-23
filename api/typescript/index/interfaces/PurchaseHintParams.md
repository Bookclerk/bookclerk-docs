[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PurchaseHintParams

# Interface: PurchaseHintParams

Defined in: [generated.ts:1808](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1808)

Params of `ContentSource.purchaseHint`. At least one identity field
(`productId` / `asin` / `isbn` / title+authors) should be set; guests may
return `invalid_params` when none are usable.

## Properties

### asin?

> `optional` **asin?**: `string`

Defined in: [generated.ts:1828](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1828)

Amazon ASIN when known.
Omitted when absent (wire zero value).

***

### authors?

> `optional` **authors?**: `string`

Defined in: [generated.ts:1823](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1823)

Authors string for fuzzy lookup.
Omitted when absent (wire zero value).

***

### isbn?

> `optional` **isbn?**: `string`

Defined in: [generated.ts:1833](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1833)

ISBN when known.
Omitted when absent (wire zero value).

***

### productId?

> `optional` **productId?**: `string`

Defined in: [generated.ts:1813](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1813)

Storefront product id when known.
Omitted when absent (wire zero value).

***

### region

> **region**: `string`

Defined in: [generated.ts:1835](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1835)

Storefront region / marketplace code.

***

### title?

> `optional` **title?**: `string`

Defined in: [generated.ts:1818](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1818)

Title text for fuzzy lookup.
Omitted when absent (wire zero value).

***

### withPrice

> **withPrice**: `boolean`

Defined in: [generated.ts:1837](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1837)

When true, guests should include live price fields when available.
