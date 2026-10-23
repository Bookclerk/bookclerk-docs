[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / D1Meta

# Interface: D1Meta

Defined in: [db-execute.ts:150](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L150)

Cloudflare [D1Result.meta](https://developers.cloudflare.com/d1/worker-api/return-object/)
shape (Bookclerk fills fields available from the typed execute reply).

## Properties

### changed\_db

> **changed\_db**: `boolean`

Defined in: [db-execute.ts:158](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L158)

`true` when `changes > 0`.

***

### changes

> **changes**: `number`

Defined in: [db-execute.ts:154](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L154)

Rows changed by the statement (`rowsAffected`).

***

### duration

> **duration**: `number`

Defined in: [db-execute.ts:152](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L152)

Engine-reported SQL duration in milliseconds.

***

### last\_row\_id

> **last\_row\_id**: `number`

Defined in: [db-execute.ts:156](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L156)

Last inserted row id when the adapter exposes it (else `0`).

***

### rows\_read

> **rows\_read**: `number`

Defined in: [db-execute.ts:160](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L160)

Rows returned to the guest for this statement.

***

### rows\_written

> **rows\_written**: `number`

Defined in: [db-execute.ts:162](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L162)

Rows written (`changes` for DML).
