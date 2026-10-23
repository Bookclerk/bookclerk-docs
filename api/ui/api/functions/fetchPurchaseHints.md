[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchPurchaseHints

# Function: fetchPurchaseHints()

> **fetchPurchaseHints**(`body`): `Promise`\<[`PurchaseHintsResponse`](../interfaces/PurchaseHintsResponse.md)\>

Defined in: [api.ts:3438](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3438)

Resolves purchase hints, coalescing viewport-gated cards into a short batch window.

## Parameters

### body

[`PurchaseHintsQuery`](../interfaces/PurchaseHintsQuery.md)

Title identity and optional candidate edition.

## Returns

`Promise`\<[`PurchaseHintsResponse`](../interfaces/PurchaseHintsResponse.md)\>

Hints list plus optional best pick (client-cached).
