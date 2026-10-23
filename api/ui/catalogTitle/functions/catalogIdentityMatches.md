[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / catalogIdentityMatches

# Function: catalogIdentityMatches()

> **catalogIdentityMatches**(`a`, `b`): `boolean`

Defined in: [catalogTitle.ts:713](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L713)

True when two catalog identities refer to the same work (work key, ASIN, ISBN, or title).

## Parameters

### a

First identity.

#### asin?

`string` \| `null`

#### isbn?

`string` \| `null`

#### title?

`string` \| `null`

#### work_key?

`string` \| `null`

### b

Second identity.

#### asin?

`string` \| `null`

#### isbn?

`string` \| `null`

#### title?

`string` \| `null`

#### work_key?

`string` \| `null`

## Returns

`boolean`

Whether the rows should share store editions and wishlist state.
