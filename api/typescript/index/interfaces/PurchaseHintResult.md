[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PurchaseHintResult

# Interface: PurchaseHintResult

Defined in: [generated.ts:2051](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2051)

Success payload of `ContentSource.purchaseHint`.

## Properties

### found

> **found**: `boolean`

Defined in: [generated.ts:2053](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2053)

False when the guest could not resolve the title (`hint` is empty).

***

### hint

> **hint**: [`PurchaseHint`](PurchaseHint.md)

Defined in: [generated.ts:2055](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2055)

Purchase hint when `found`.
