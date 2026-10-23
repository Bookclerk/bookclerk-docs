[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ProgressSink

# Class: ProgressSink

Defined in: [plugin.ts:387](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L387)

Progress reports for a job invocation (never carries media).

## Extends

- `RpcTarget`

## Constructors

### Constructor

> **new ProgressSink**(): `ProgressSink`

#### Returns

`ProgressSink`

#### Inherited from

`RpcTarget.constructor`

## Methods

### report()

> **report**(`_percent`, `_message`): `Promise`\<`void`\>

Defined in: [plugin.ts:395](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L395)

Reports `percent` in `0..=100` and an operator-facing `message`.

#### Parameters

##### \_percent

`number`

Completion percent.

##### \_message

`string`

Operator-facing status.

#### Returns

`Promise`\<`void`\>

Resolves when the host records progress.
