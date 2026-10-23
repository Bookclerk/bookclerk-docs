[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / TypedDbStatement

# Interface: TypedDbStatement

Defined in: [db-execute.ts:42](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L42)

One statement in a typed atomic batch.

## Properties

### kind

> **kind**: `"execute"` \| `"select"` \| `"returning"`

Defined in: [db-execute.ts:48](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L48)

Host-authored kind (adapters must not reparse SQL).

***

### maxRows

> **maxRows**: `number`

Defined in: [db-execute.ts:50](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L50)

Proven row upper bound (`0` = unproven).

***

### parameters

> **parameters**: [`DbValue`](../type-aliases/DbValue.md)[]

Defined in: [db-execute.ts:46](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L46)

Ordered typed binds.

***

### resultSelection

> **resultSelection**: `"discard"` \| `"affectedRows"` \| `"rows"`

Defined in: [db-execute.ts:52](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L52)

Which result fields the caller needs.

***

### sql

> **sql**: `string`

Defined in: [db-execute.ts:44](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L44)

Canonical Bookclerk SQL (`?` placeholders).
