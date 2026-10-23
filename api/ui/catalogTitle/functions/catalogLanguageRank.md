[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / catalogLanguageRank

# Function: catalogLanguageRank()

> **catalogLanguageRank**(`hitLanguage`, `preferred?`): `number`

Defined in: [catalogTitle.ts:423](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L423)

Soft language rank for ordering search hits.

## Parameters

### hitLanguage

`string` \| `null` \| `undefined`

Language on the hit.

### preferred?

`string` = `...`

Preferred language (default: browser).

## Returns

`number`

`0` preferred, `1` unknown, `2` other.
