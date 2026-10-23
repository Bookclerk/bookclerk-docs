[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / pickBetterDescription

# Function: pickBetterDescription()

> **pickBetterDescription**(`a`, `b`): `string` \| `null`

Defined in: [catalogTitle.ts:884](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L884)

Prefers richer store/Audnexus blurbs over short plain catalog teasers.

HTML descriptions win over plain text; otherwise the longer string wins.

## Parameters

### a

`string` \| `null` \| `undefined`

Current description.

### b

`string` \| `null` \| `undefined`

Incoming description.

## Returns

`string` \| `null`

Preferred non-empty description, or `null`.
