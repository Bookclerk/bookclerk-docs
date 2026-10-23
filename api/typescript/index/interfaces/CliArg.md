[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CliArg

# Interface: CliArg

Defined in: [generated.ts:1274](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1274)

One named argument value passed to `PluginCli.invoke`.

## Properties

### name

> **name**: `string`

Defined in: [generated.ts:1276](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1276)

Arg name matching a `CliArgSpec.name`.

***

### value

> **value**: `string`

Defined in: [generated.ts:1278](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1278)

String form of the value (the guest parses per `CliArgSpec.kind`).
