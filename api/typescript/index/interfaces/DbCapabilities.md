[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DbCapabilities

# Interface: DbCapabilities

Defined in: [generated.ts:2684](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2684)

Semantic SQL-contract advertisement. Diagnostic engine identity is not
part of the capability plane — see `DbBootstrap`.

## Properties

### affectedRows

> **affectedRows**: `boolean`

Defined in: [generated.ts:2692](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2692)

Guest reports `rowsAffected`.

***

### atomicBatch

> **atomicBatch**: `boolean`

Defined in: [generated.ts:2688](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2688)

Guest can run a bounded statement list as one SQL transaction.

***

### atomicUnitRestore

> **atomicUnitRestore**: `boolean`

Defined in: [generated.ts:2753](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2753)

Adapter can destructively replace one logical database unit so an
ordinary restore failure does not leave that unit partially replaced.

***

### cancellation

> **cancellation**: `boolean`

Defined in: [generated.ts:2696](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2696)

Guest honors RPC/session cancellation.

***

### consistentBackupRead

> **consistentBackupRead**: `boolean`

Defined in: [generated.ts:2748](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2748)

Adapter can expose one stable logical database state while the host
reads schema, rows, and identity.

***

### maxAtomicResultBytes

> **maxAtomicResultBytes**: `number`

Defined in: [generated.ts:2714](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2714)

Maximum encoded bytes of one `ExecuteReply`.

***

### maxBinds

> **maxBinds**: `number`

Defined in: [generated.ts:2700](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2700)

Maximum bound parameters per statement.

***

### maxCellBytes

> **maxCellBytes**: `number`

Defined in: [generated.ts:2710](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2710)

Maximum UTF-8 / blob bytes of one result cell.

***

### maxFunctionArgs

> **maxFunctionArgs**: `number`

Defined in: [generated.ts:2725](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2725)

Maximum arguments in one physical function call after adapter hiding
(nested min / max / coalesce). Portable json_object is already ≤ 32
arguments (16 pairs), matching D1. `0` is unspecified.

***

### maxLoweredStatementBytes

> **maxLoweredStatementBytes**: `number`

Defined in: [generated.ts:2743](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2743)

Maximum UTF-8 bytes of sqlite-family lowered SQL the adapter can realize
for one statement (INTEGER overflow wraps, LIKE→GLOB, NULLIF, INSERT OR
IGNORE, query LIMIT wrap, bytes-placeholder expansion). `0` is
unspecified: the host does not enforce a lowered-size ceiling. First-party
D1 advertises 100000. Hosts compare a standardized Bookclerk lowering
upper bound against this number and must not branch on engine identity.

***

### maxPatternBytes

> **maxPatternBytes**: `number`

Defined in: [generated.ts:2734](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2734)

Maximum UTF-8 bytes of a BookclerkSQL LIKE pattern value (literals and
TEXT binds). Adapters that expand LIKE into GLOB must advertise a
conservative value that still fits the physical pattern cap. `0` is
unspecified.

***

### maxPayloadBytes

> **maxPayloadBytes**: `number`

Defined in: [generated.ts:2706](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2706)

Maximum UTF-8 bytes of SQL plus binds per statement.

***

### maxRequestBytes

> **maxRequestBytes**: `number`

Defined in: [generated.ts:2712](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2712)

Maximum encoded bytes of one `ExecuteRequest`.

***

### maxResultBytes

> **maxResultBytes**: `number`

Defined in: [generated.ts:2708](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2708)

Maximum encoded bytes of one statement's result rows.

***

### maxResultRows

> **maxResultRows**: `number`

Defined in: [generated.ts:2704](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2704)

Maximum rows a query statement may return.

***

### maxSchemaColumns

> **maxSchemaColumns**: `number`

Defined in: [generated.ts:2727](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2727)

Maximum columns in one CREATE TABLE / result row. `0` is unspecified.

***

### maxStatements

> **maxStatements**: `number`

Defined in: [generated.ts:2702](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2702)

Maximum statements in one atomic batch.

***

### pluginDatabases

> **pluginDatabases**: `boolean`

Defined in: [generated.ts:2719](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2719)

Append-only. Adapter can open additional isolated sessions
for plugin-owned database bindings (per-binding file / schema / database).

***

### returning

> **returning**: `boolean`

Defined in: [generated.ts:2690](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2690)

Guest SQL supports `RETURNING`.

***

### schemaMigrations

> **schemaMigrations**: `boolean`

Defined in: [generated.ts:2694](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2694)

Guest versions schema with a `bookclerk_schema_migrations` table.

***

### sqlContractVersion

> **sqlContractVersion**: `number`

Defined in: [generated.ts:2686](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2686)

Bookclerk SQL contract version.

***

### timing

> **timing**: `boolean`

Defined in: [generated.ts:2698](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2698)

Guest can fill `DbTiming.dbExecutionUs`. Not a host connect minimum.
