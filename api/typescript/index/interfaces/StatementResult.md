[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / StatementResult

# Interface: StatementResult

Defined in: [db-execute.ts:90](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L90)

Result of one statement in an atomic batch.

## Properties

### columns

> **columns**: [`DbColumn`](DbColumn.md)[]

Defined in: [db-execute.ts:94](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L94)

Column metadata aligned with each row.

***

### rows

> **rows**: [`DbRow`](DbRow.md)[]

Defined in: [db-execute.ts:92](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L92)

Positional rows.

***

### rowsAffected

> **rowsAffected**: `number`

Defined in: [db-execute.ts:96](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L96)

Engine `rowsAffected` (0 for `Select`).
