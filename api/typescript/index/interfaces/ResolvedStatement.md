[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ResolvedStatement

# Interface: ResolvedStatement

Defined in: [generated.ts:2586](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2586)

Host-produced typed proof bound to one exact canonical statement.

## Properties

### assignments

> **assignments**: [`ResolvedAssignment`](ResolvedAssignment.md)[]

Defined in: [generated.ts:2594](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2594)

Mutation destination assignments.

***

### functions

> **functions**: `string`[]

Defined in: [generated.ts:2600](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2600)

Function names invoked (folded), for authorization.

***

### integerArithSites

> **integerArithSites**: [`IntegerArithSite`](IntegerArithSite.md)[]

Defined in: [generated.ts:2598](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2598)

INTEGER overflow sites (`+` `-` `*` `abs`).

***

### outputColumns

> **outputColumns**: [`NamedSqlType`](NamedSqlType.md)[]

Defined in: [generated.ts:2590](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2590)

SELECT / RETURNING / VALUES output columns in order.

***

### physicalAccesses

> **physicalAccesses**: [`PhysicalAccess`](PhysicalAccess.md)[]

Defined in: [generated.ts:2592](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2592)

Physical tables/columns referenced (authorization).

***

### schemaAction

> **schemaAction**: [`SchemaAction`](../type-aliases/SchemaAction.md)

Defined in: [generated.ts:2602](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2602)

DDL action + fingerprint.

***

### statementHash

> **statementHash**: `string`

Defined in: [generated.ts:2588](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2588)

SHA-256 hex of the exact canonical SQL this proof claims.

***

### textCollateSites

> **textCollateSites**: [`TextCollateSite`](TextCollateSite.md)[]

Defined in: [generated.ts:2596](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2596)

TEXT expression spans needing bytewise collation.
