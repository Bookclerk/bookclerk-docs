[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Source

# Class: Source

Defined in: [plugin.ts:374](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L374)

Byte source capability: `job.input`.

## Extends

- `RpcTarget`

## Constructors

### Constructor

> **new Source**(): `Source`

#### Returns

`Source`

#### Inherited from

`RpcTarget.constructor`

## Methods

### open()

> **open**(`_key`): `Promise`\<[`ReadResult`](../interfaces/ReadResult.md)\>

Defined in: [plugin.ts:381](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L381)

Opens `key` for streamed reading.

#### Parameters

##### \_key

`string`

Object key.

#### Returns

`Promise`\<[`ReadResult`](../interfaces/ReadResult.md)\>

Metadata plus a transferred body stream.
