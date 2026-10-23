[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ClusterLinkView

# Interface: ClusterLinkView

Defined in: [api.ts:2317](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2317)

An owner cluster this install is enrolled in as a host (no key material).

## Properties

### cluster\_id

> **cluster\_id**: `string`

Defined in: [api.ts:2319](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2319)

Id of the owner cluster; one link per cluster.

***

### enrolled\_at

> **enrolled\_at**: `string`

Defined in: [api.ts:2329](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2329)

RFC 3339 time of the latest enrollment (a re-enrollment resets it).

***

### include\_branches?

> `optional` **include\_branches?**: `boolean`

Defined in: [api.ts:2333](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2333)

"Include books from my own branches" (on by default).

***

### node\_id

> **node\_id**: `string`

Defined in: [api.ts:2323](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2323)

Node id the owner minted for this install.

***

### owner\_origin

> **owner\_origin**: `string`

Defined in: [api.ts:2321](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2321)

Canonical origin (`https://host[:port]`) this host dials the owner at.

***

### pinned

> **pinned**: `boolean`

Defined in: [api.ts:2325](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2325)

True when the link pins the owner's self-signed certificate.

***

### share\_books?

> `optional` **share\_books?**: `boolean`

Defined in: [api.ts:2331](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2331)

"Share this branch's books with <library>" (on by default).

***

### state

> **state**: `string`

Defined in: [api.ts:2327](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2327)

`active`, `revoked` (by the owner), or `left` (by this host).
