[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / signOut

# Function: signOut()

> **signOut**(`role?`): `Promise`\<`void`\>

Defined in: [api.ts:2748](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2748)

Signs out regardless of operator vs user/portal session.

## Parameters

### role?

[`AuthRole`](../type-aliases/AuthRole.md)

Current session role; portal roles use portal logout.

## Returns

`Promise`\<`void`\>

Resolves when logout completes.
