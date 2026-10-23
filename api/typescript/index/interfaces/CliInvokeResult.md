[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CliInvokeResult

# Interface: CliInvokeResult

Defined in: [generated.ts:1290](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1290)

Result of `PluginCli.invoke`.

## Properties

### exitCode

> **exitCode**: `number`

Defined in: [generated.ts:1292](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1292)

Process-style exit code (0 = success).

***

### payload

> **payload**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [generated.ts:1298](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1298)

Structured payload for machine consumers; empty `mediaType` when absent.

***

### stderr

> **stderr**: `string`

Defined in: [generated.ts:1296](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1296)

Captured standard error text.

***

### stdout

> **stdout**: `string`

Defined in: [generated.ts:1294](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1294)

Captured standard output text.
