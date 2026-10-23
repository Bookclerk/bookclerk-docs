[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / GuestDatabase

# Interface: GuestDatabase

Defined in: [generated.ts:2886](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2886)

Host-granted SQL for job plugin authors. SDK `DatabaseBinding` mirrors the
Cloudflare Workers D1 surface (`prepare`/`bind`/`run`/`first`/`all`/`raw`,
`batch`, `exec`) over this typed `execute` transport; wire types stay Cap'n
`ExecuteRequest`/`ExecuteReply`.

## Methods

### close()

> **close**(): `Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

Defined in: [generated.ts:2899](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2899)

Release the session; further calls fail.

#### Returns

`Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

[EmptyReply](../type-aliases/EmptyReply.md)

***

### execute()

> **execute**(`request`): `Promise`\<[`ExecuteResultReply`](../type-aliases/ExecuteResultReply.md)\>

Defined in: [generated.ts:2893](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2893)

Run one typed statement batch.

#### Parameters

##### request

`ExecuteRequest`

Statements, binds, and deadline.

#### Returns

`Promise`\<[`ExecuteResultReply`](../type-aliases/ExecuteResultReply.md)\>

[ExecuteResultReply](../type-aliases/ExecuteResultReply.md)
