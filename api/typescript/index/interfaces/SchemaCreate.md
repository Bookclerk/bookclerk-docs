[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / SchemaCreate

# Interface: SchemaCreate

Defined in: [generated.ts:2570](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2570)

`CREATE TABLE` action with its durable fingerprint.

## Properties

### fingerprint

> **fingerprint**: `string`

Defined in: [generated.ts:2574](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2574)

Structured schema fingerprint (hex SHA-256).

***

### noop

> **noop**: `boolean`

Defined in: [generated.ts:2576](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2576)

True when the catalog already holds this exact fingerprint.

***

### schema

> **schema**: [`CreateTableSchema`](CreateTableSchema.md)

Defined in: [generated.ts:2572](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2572)

Parsed table schema.
