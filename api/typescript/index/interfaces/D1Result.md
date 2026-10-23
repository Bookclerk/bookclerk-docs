[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / D1Result

# Interface: D1Result\<T\>

Defined in: [db-execute.ts:169](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L169)

Cloudflare [D1Result](https://developers.cloudflare.com/d1/worker-api/return-object/)
projection for plugin guests. Errors throw; successful calls always set `success: true`.

## Type Parameters

### T

`T` = `Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>

## Properties

### meta

> **meta**: [`D1Meta`](D1Meta.md)

Defined in: [db-execute.ts:173](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L173)

***

### results

> **results**: `T`[] \| `null`

Defined in: [db-execute.ts:172](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L172)

Row objects for selects; `[]` when empty; `null` when not applicable (DML).

***

### success

> **success**: `true`

Defined in: [db-execute.ts:170](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L170)
