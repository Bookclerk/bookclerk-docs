[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / DiscoverSearchHandoff

# Type Alias: DiscoverSearchHandoff

> **DiscoverSearchHandoff** = `object`

Defined in: [routes.ts:204](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L204)

Optional post-search filter/sort applied after a Discover catalog query.

## Properties

### authors?

> `optional` **authors?**: `string`[]

Defined in: [routes.ts:218](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L218)

Seed include filters (server-applied).

***

### excludeSources?

> `optional` **excludeSources?**: `string`[]

Defined in: [routes.ts:223](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L223)

***

### field?

> `optional` **field?**: `"author"` \| `"narrator"` \| `"series"` \| `"genre"`

Defined in: [routes.ts:207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L207)

Storefront-scoped search (`author` / `narrator` / `series` / `genre`).

***

### ~~filterKind?~~

> `optional` **filterKind?**: `"all"` \| `"authors"` \| `"series"` \| `"narrators"` \| `"genres"` \| `"sources"`

Defined in: [routes.ts:234](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L234)

#### Deprecated

legacy single-dimension filter

***

### ~~filterValue?~~

> `optional` **filterValue?**: `string`

Defined in: [routes.ts:242](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L242)

#### Deprecated

***

### genres?

> `optional` **genres?**: `string`[]

Defined in: [routes.ts:221](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L221)

***

### hideVirtualVoice?

> `optional` **hideVirtualVoice?**: `boolean`

Defined in: [routes.ts:232](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L232)

When false, do not exclude Virtual Voice (default hide is on).

***

### language?

> `optional` **language?**: `string`

Defined in: [routes.ts:230](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L230)

Hard language include (`en`, `zh`, …) or `__all__` for no hard filter.
Omit to use the browser language default.

***

### minRating?

> `optional` **minRating?**: `number`

Defined in: [routes.ts:224](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L224)

***

### narrators?

> `optional` **narrators?**: `string`[]

Defined in: [routes.ts:219](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L219)

***

### q

> **q**: `string`

Defined in: [routes.ts:205](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L205)

***

### runtimeBucket?

> `optional` **runtimeBucket?**: `"any"` \| `"under6"` \| `"6to12"` \| `"12to20"` \| `"over20"`

Defined in: [routes.ts:225](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L225)

***

### series?

> `optional` **series?**: `string`[]

Defined in: [routes.ts:220](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L220)

***

### sort?

> `optional` **sort?**: `"relevance"` \| `"popularity"` \| `"rating"` \| `"title"` \| `"author"` \| `"price"` \| `"length"`

Defined in: [routes.ts:208](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L208)

***

### sortDir?

> `optional` **sortDir?**: `"asc"` \| `"desc"`

Defined in: [routes.ts:216](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L216)

***

### ~~sortKey?~~

> `optional` **sortKey?**: `"title"` \| `"author"` \| `"series"` \| `"relevance"` \| `"popularity"` \| `"rating"` \| `"price"` \| `"length"`

Defined in: [routes.ts:244](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L244)

#### Deprecated

***

### sources?

> `optional` **sources?**: `string`[]

Defined in: [routes.ts:222](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L222)
