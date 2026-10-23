[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / RemoteConsentView

# Interface: RemoteConsentView

Defined in: [api.ts:2389](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2389)

One owner plugin's request to run on this host, and this host's answer.

## Properties

### approved\_at

> **approved\_at**: `string` \| `null`

Defined in: [api.ts:2414](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2414)

RFC 3339 time of the approval, if approved.

***

### approved\_by

> **approved\_by**: `string` \| `null`

Defined in: [api.ts:2416](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2416)

Who approved: the operator's audit identity, or `platform` for an automatic approval.

***

### approved\_version

> **approved\_version**: `string` \| `null`

Defined in: [api.ts:2410](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2410)

Version this host approved, if any.

***

### cluster\_id

> **cluster\_id**: `string`

Defined in: [api.ts:2391](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2391)

Owner cluster that asks; matches a [ClusterLinkView.cluster\_id](ClusterLinkView.md#cluster_id).

***

### grant

> **grant**: [`RemoteGrantView`](RemoteGrantView.md) \| `null`

Defined in: [api.ts:2412](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2412)

What this host granted (the request, narrowed), if approved.

***

### plugin\_key

> **plugin\_key**: `string`

Defined in: [api.ts:2393](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2393)

Canonical plugin key the request is for.

***

### request\_digest

> **request\_digest**: `string`

Defined in: [api.ts:2408](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2408)

Digest of the whole request (version, grant, archive); approve by it.

***

### requested\_artifact?

> `optional` **requested\_artifact?**: `object`

Defined in: [api.ts:2399](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2399)

The archive this host would fetch for that version (hex SHA-256 digests).

#### archive\_sha256

> **archive\_sha256**: `string`

Digest of the archive file.

#### payload\_root\_sha256

> **payload\_root\_sha256**: `string`

Digest of the unpacked payload tree.

#### url

> **url**: `string`

`blob:` (served by the owner) or `https:` archive URL.

***

### requested\_grant

> **requested\_grant**: [`RemoteGrantView`](RemoteGrantView.md) \| `null`

Defined in: [api.ts:2397](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2397)

What that version asks for; `null` when the stored request does not parse.

***

### requested\_version

> **requested\_version**: `string`

Defined in: [api.ts:2395](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2395)

Version the owner now asks to run.

***

### revoked\_at?

> `optional` **revoked\_at?**: `string` \| `null`

Defined in: [api.ts:2418](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2418)

RFC 3339 time this host's operator revoked it; a revoke holds until approved again.

***

### state

> **state**: [`RemoteConsentState`](../type-aliases/RemoteConsentState.md)

Defined in: [api.ts:2422](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2422)

Whether the guest may run here now, or what it waits on.

***

### updated\_at?

> `optional` **updated\_at?**: `string`

Defined in: [api.ts:2420](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2420)

RFC 3339 time the row last changed.
