[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CatalogDetail

# Interface: CatalogDetail

Defined in: [generated.ts:1987](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1987)

Success payload of `ContentSource.catalogDetail`.

## Properties

### found

> **found**: `boolean`

Defined in: [generated.ts:1989](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1989)

False when the product is unknown to the storefront (`hit` is empty).

***

### hit

> **hit**: [`CatalogHit`](CatalogHit.md)

Defined in: [generated.ts:1991](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1991)

Full catalog record when `found`.
