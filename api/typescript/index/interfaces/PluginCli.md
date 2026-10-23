[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginCli

# Interface: PluginCli

Defined in: [generated.ts:1107](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1107)

`cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`.

## Methods

### describe()

> **describe**(): `Promise`\<[`CliSchemaReply`](../type-aliases/CliSchemaReply.md)\>

Defined in: [generated.ts:1113](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1113)

Declared CLI surface.

#### Returns

`Promise`\<[`CliSchemaReply`](../type-aliases/CliSchemaReply.md)\>

[CliSchemaReply](../type-aliases/CliSchemaReply.md)

***

### invoke()

> **invoke**(`params`): `Promise`\<[`CliInvokeReply`](../type-aliases/CliInvokeReply.md)\>

Defined in: [generated.ts:1120](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1120)

Run one plugin CLI command.

#### Parameters

##### params

[`CliInvokeParams`](CliInvokeParams.md)

Command name and argument values.

#### Returns

`Promise`\<[`CliInvokeReply`](../type-aliases/CliInvokeReply.md)\>

[CliInvokeReply](../type-aliases/CliInvokeReply.md)
