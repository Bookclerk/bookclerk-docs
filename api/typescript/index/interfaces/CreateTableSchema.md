[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CreateTableSchema

# Interface: CreateTableSchema

Defined in: [generated.ts:2546](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2546)

Parsed `CREATE TABLE` (canonical SQL v1). Per-column lists align with `columns`.

## Properties

### columnChecks

> **columnChecks**: `string`[]

Defined in: [generated.ts:2562](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2562)

Per-column CHECK expression text; empty when none.

***

### columnDefaults

> **columnDefaults**: `string`[]

Defined in: [generated.ts:2560](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2560)

Per-column DEFAULT expression text; empty when none.

***

### columnNotNull

> **columnNotNull**: `boolean`[]

Defined in: [generated.ts:2554](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2554)

Per-column NOT NULL flags.

***

### columnPrimaryKey

> **columnPrimaryKey**: `boolean`[]

Defined in: [generated.ts:2558](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2558)

Per-column PRIMARY KEY flags.

***

### columnReferences

> **columnReferences**: [`OptionalColumnReference`](../type-aliases/OptionalColumnReference.md)[]

Defined in: [generated.ts:2564](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2564)

Per-column REFERENCES targets.

***

### columns

> **columns**: [`NamedSqlType`](NamedSqlType.md)[]

Defined in: [generated.ts:2550](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2550)

Columns with resolved types, in order.

***

### columnUnique

> **columnUnique**: `boolean`[]

Defined in: [generated.ts:2556](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2556)

Per-column UNIQUE flags.

***

### identityColumn

> **identityColumn**: `string`

Defined in: [generated.ts:2552](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2552)

`INTEGER PRIMARY KEY AUTOINCREMENT` column; empty when none.

***

### table

> **table**: `string`

Defined in: [generated.ts:2548](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2548)

Table name.

***

### tableConstraints

> **tableConstraints**: [`TableConstraint`](../type-aliases/TableConstraint.md)[]

Defined in: [generated.ts:2566](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2566)

Table-level constraints, in order.
