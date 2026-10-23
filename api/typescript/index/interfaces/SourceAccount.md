[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / SourceAccount

# Interface: SourceAccount

Defined in: [generated.ts:1360](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1360)

Source account metadata returned from login and stored by the host.

## Properties

### accountId

> **accountId**: `string`

Defined in: [generated.ts:1362](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1362)

Stable account id within this source plugin.

***

### label?

> `optional` **label?**: `string`

Defined in: [generated.ts:1371](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1371)

Operator-facing label.
Omitted when absent (wire zero value).

***

### marketplace

> **marketplace**: `string`

Defined in: [generated.ts:1366](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1366)

Storefront marketplace / region code (for example `us`, `uk`).

***

### scanEnabled

> **scanEnabled**: `boolean`

Defined in: [generated.ts:1376](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1376)

When true, bare / scheduled scans include this account. Explicit CLI
`--account` bypasses this flag.

***

### source

> **source**: `string`

Defined in: [generated.ts:1364](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1364)

Source plugin id (the host forces this to the guest's install id).
