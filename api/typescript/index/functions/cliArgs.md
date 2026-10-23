[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / cliArgs

# Function: cliArgs()

> **cliArgs**(`params`): `Record`\<`string`, `string`\>

Defined in: [plugin.ts:653](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L653)

Turns `CliInvokeParams.args` (`[{ name, value }]`) into a plain
`{ name: value }` object for CLI handlers.

## Parameters

### params

`Pick`\<[`CliInvokeParams`](../interfaces/CliInvokeParams.md), `"args"`\> \| `undefined`

CLI invocation params from the host.

## Returns

`Record`\<`string`, `string`\>

Argument values keyed by name.
