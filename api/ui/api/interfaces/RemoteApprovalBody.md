[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / RemoteApprovalBody

# Interface: RemoteApprovalBody

Defined in: [api.ts:2437](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2437)

Body of an approval: the reviewed request and what of it to leave out.
An approval can only narrow the request, never widen it.

## Properties

### cluster\_id

> **cluster\_id**: `string`

Defined in: [api.ts:2439](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2439)

Owner cluster of the request.

***

### deny\_bindings?

> `optional` **deny\_bindings?**: `string`[]

Defined in: [api.ts:2447](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2447)

Host bindings to remove (`secrets`, `oauth`, `kv`, …).

***

### deny\_domains?

> `optional` **deny\_domains?**: `string`[]

Defined in: [api.ts:2445](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2445)

Outbound domain patterns to remove from the grant.

***

### deny\_network?

> `optional` **deny\_network?**: `boolean`

Defined in: [api.ts:2449](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2449)

True turns outbound network off altogether (no domains, TCP or CIDRs).

***

### plugin\_key

> **plugin\_key**: `string`

Defined in: [api.ts:2441](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2441)

Plugin key of the request.

***

### request\_digest

> **request\_digest**: `string`

Defined in: [api.ts:2443](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2443)

The `request_digest` reviewed; refused when the recorded request is another one.
