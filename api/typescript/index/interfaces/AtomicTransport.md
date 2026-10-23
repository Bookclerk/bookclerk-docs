[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / AtomicTransport

# Interface: AtomicTransport

Defined in: [db-execute.ts:126](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L126)

Host `GuestDatabase.execute` projection.

## Extended by

- [`GrantedDatabaseTransport`](GrantedDatabaseTransport.md)

## Methods

### execute()

> **execute**(`request`): `Promise`\<[`ExecuteReply`](ExecuteReply.md)\>

Defined in: [db-execute.ts:133](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L133)

Runs one typed atomic batch.

#### Parameters

##### request

[`ExecuteRequest`](ExecuteRequest.md)

Structured Cap'n `ExecuteRequest`.

#### Returns

`Promise`\<[`ExecuteReply`](ExecuteReply.md)\>

Structured `ExecuteReply`.
