[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / IdentityHighWater

# Interface: IdentityHighWater

Defined in: [generated.ts:2796](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2796)

Adapter-private identity high-water (sqlite_sequence / bookclerk_identity).
Column names live in the canonical backup schema, not this catalog.

## Properties

### last

> **last**: `bigint`

Defined in: [generated.ts:2800](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2800)

Highest generated or stored value that must not be reused.

***

### table

> **table**: `string`

Defined in: [generated.ts:2798](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2798)

Table whose identity column the mark belongs to.
