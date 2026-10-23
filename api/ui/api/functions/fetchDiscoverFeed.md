[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchDiscoverFeed

# Function: fetchDiscoverFeed()

> **fetchDiscoverFeed**(`limit?`): `Promise`\<[`DiscoverFeed`](../interfaces/DiscoverFeed.md)\>

Defined in: [api.ts:3229](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L3229)

Loads Discover recommendation shelves.

## Parameters

### limit?

`number` = `36`

Max items per shelf (default 36). Seed URLs only; live prices load later.

## Returns

`Promise`\<[`DiscoverFeed`](../interfaces/DiscoverFeed.md)\>

Discover feed payload.
