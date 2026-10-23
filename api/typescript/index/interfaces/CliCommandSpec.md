[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CliCommandSpec

# Interface: CliCommandSpec

Defined in: [generated.ts:1229](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1229)

One plugin CLI command under `CliSchema`.

## Properties

### about?

> `optional` **about?**: `string`

Defined in: [generated.ts:1236](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1236)

Short help text for `--help`.
Omitted when absent (wire zero value).

***

### args

> **args**: [`CliArgSpec`](CliArgSpec.md)[]

Defined in: [generated.ts:1238](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1238)

Argument / flag specs for this command.

***

### name

> **name**: `string`

Defined in: [generated.ts:1231](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1231)

Command verb after the plugin id (for example "ping").
