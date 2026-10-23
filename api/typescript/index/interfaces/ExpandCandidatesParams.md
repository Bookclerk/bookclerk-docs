[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ExpandCandidatesParams

# Interface: ExpandCandidatesParams

Defined in: [generated.ts:1760](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1760)

Params of `ContentSource.expandCandidates`. Seed fields identify a known
title; the guest returns related catalog hits.

## Properties

### asin?

> `optional` **asin?**: `string`

Defined in: [generated.ts:1791](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1791)

Seed Amazon ASIN.
Omitted when absent (wire zero value).

***

### authors?

> `optional` **authors?**: `string`

Defined in: [generated.ts:1771](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1771)

Seed authors string.
Omitted when absent (wire zero value).

***

### isbn?

> `optional` **isbn?**: `string`

Defined in: [generated.ts:1796](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1796)

Seed ISBN.
Omitted when absent (wire zero value).

***

### limit

> **limit**: `number`

Defined in: [generated.ts:1800](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1800)

Maximum candidates to return; the host always sends an explicit value.

***

### narrators?

> `optional` **narrators?**: `string`

Defined in: [generated.ts:1776](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1776)

Seed narrators string.
Omitted when absent (wire zero value).

***

### productId

> **productId**: `string`

Defined in: [generated.ts:1764](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1764)

Seed storefront product id.

***

### region

> **region**: `string`

Defined in: [generated.ts:1798](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1798)

Storefront region / marketplace code.

***

### series?

> `optional` **series?**: `string`

Defined in: [generated.ts:1781](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1781)

Seed series name.
Omitted when absent (wire zero value).

***

### seriesAsin?

> `optional` **seriesAsin?**: `string`

Defined in: [generated.ts:1786](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1786)

Seed series ASIN when known.
Omitted when absent (wire zero value).

***

### source

> **source**: `string`

Defined in: [generated.ts:1762](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1762)

Source plugin id hint when expanding across storefronts.

***

### title

> **title**: `string`

Defined in: [generated.ts:1766](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1766)

Seed title text.
