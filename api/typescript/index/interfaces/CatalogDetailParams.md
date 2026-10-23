[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CatalogDetailParams

# Interface: CatalogDetailParams

Defined in: [generated.ts:1850](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1850)

Params of `ContentSource.catalogDetail`.

## Properties

### isbn?

> `optional` **isbn?**: `string`

Defined in: [generated.ts:1857](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1857)

ISBN when it differs from `productId`.
Omitted when absent (wire zero value).

***

### productId

> **productId**: `string`

Defined in: [generated.ts:1852](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1852)

Store product id (Libro ISBN or ISBN-slug).
