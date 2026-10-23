[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobRunner

# Interface: JobRunner

Defined in: [generated.ts:808](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L808)

`[triggers] jobs` handler (`Entrypoints.jobRunner`); runs one durable command.

## Methods

### job()

> **job**(`controller`): `Promise`\<[`HandleReply`](../type-aliases/HandleReply.md)\>

Defined in: [generated.ts:816](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L816)

Run one command invocation to a terminal or suspended outcome. Named
database bindings come from `Bindings.databases` at `open`, not per job.

#### Parameters

##### controller

`JobController`

Envelope plus host-served capabilities.

#### Returns

`Promise`\<[`HandleReply`](../type-aliases/HandleReply.md)\>

[HandleReply](../type-aliases/HandleReply.md)
