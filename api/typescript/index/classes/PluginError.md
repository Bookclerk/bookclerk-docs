[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginError

# Class: PluginError

Defined in: [errors.ts:14](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L14)

Thrown by the SDK when a wire union carries `err`. Unknown codes are kept.

## Extends

- `Error`

## Extended by

- [`InvokeError`](../../workerd/classes/InvokeError.md)

## Constructors

### Constructor

> **new PluginError**(`code`, `message`): `PluginError`

Defined in: [errors.ts:20](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L20)

#### Parameters

##### code

`string`

##### message

`string`

#### Returns

`PluginError`

#### Overrides

`Error.constructor`

## Properties

### code

> `readonly` **code**: `string`

Defined in: [errors.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L16)

Known `PluginErrorCode` wire string, or `unknown`.

***

### wireCode

> `readonly` **wireCode**: `string`

Defined in: [errors.ts:18](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L18)

Raw wire code, including codes this SDK does not know.

## Methods

### fromWire()

> `static` **fromWire**(`code`, `message`): `PluginError`

Defined in: [errors.ts:37](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L37)

Construct a PluginError from a Cap'n Proto / JSON wire code.

Unknown codes become `unknown` on [PluginError.code](#code) while
[PluginError.wireCode](#wirecode) keeps the raw value.

#### Parameters

##### code

`string`

Wire error code (known or unknown).

##### message

`string`

Operator-facing error text.

#### Returns

`PluginError`

Typed plugin error.
