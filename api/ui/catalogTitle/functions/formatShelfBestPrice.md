[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / formatShelfBestPrice

# Function: formatShelfBestPrice()

> **formatShelfBestPrice**(`hint`): `string` \| `null`

Defined in: [catalogTitle.ts:525](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L525)

Single best price for shelf cards (member / primary only — no dual list).

## Parameters

### hint

Price label fields from a purchase hint.

#### list_price_label?

`string` \| `null`

#### member_price_label?

`string` \| `null`

#### price_label?

`string` \| `null`

## Returns

`string` \| `null`

Display string, or `null` when no prices.
