[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchMfaPolicy

# Function: fetchMfaPolicy()

> **fetchMfaPolicy**(): `Promise`\<\{ `require_second_factor`: `boolean`; \}\>

Defined in: [api.ts:1545](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1545)

Host-wide password second-factor policy (`daemon.auth.require_second_factor`).

## Returns

`Promise`\<\{ `require_second_factor`: `boolean`; \}\>

Whether password login requires a second factor.
