[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / queueDiscoverSearch

# Function: queueDiscoverSearch()

> **queueDiscoverSearch**(`handoff`): `void`

Defined in: [routes.ts:260](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L260)

Queues a Discover catalog search for the next Discover page mount (cross-view).

## Parameters

### handoff

`string` \| [`DiscoverSearchHandoff`](../type-aliases/DiscoverSearchHandoff.md)

Query string or full handoff object (ignored when `q` is under 2 chars).

## Returns

`void`
