[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [workerd](../README.md) / InvokeError

# Class: InvokeError

Defined in: [invoke.ts:112](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L112)

**`Internal`**

Transport-level `/invoke` failure (never an author failure): unknown
interface/method, malformed message, bad capability table, or a missing
binding.

## Extends

- [`PluginError`](../../index/classes/PluginError.md)

## Constructors

### Constructor

> **new InvokeError**(`status`, `code`, `message`): `InvokeError`

Defined in: [invoke.ts:116](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L116)

#### Parameters

##### status

`number`

##### code

`string`

##### message

`string`

#### Returns

`InvokeError`

#### Overrides

[`PluginError`](../../index/classes/PluginError.md).[`constructor`](../../index/classes/PluginError.md#constructor)

## Properties

### code

> `readonly` **code**: `string`

Defined in: [errors.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L16)

Known `PluginErrorCode` wire string, or `unknown`.

#### Inherited from

[`PluginError`](../../index/classes/PluginError.md).[`code`](../../index/classes/PluginError.md#code)

***

### status

> `readonly` **status**: `number`

Defined in: [invoke.ts:114](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L114)

HTTP status the bridge answers with.

***

### wireCode

> `readonly` **wireCode**: `string`

Defined in: [errors.ts:18](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L18)

Raw wire code, including codes this SDK does not know.

#### Inherited from

[`PluginError`](../../index/classes/PluginError.md).[`wireCode`](../../index/classes/PluginError.md#wirecode)

## Methods

### fromWire()

> `static` **fromWire**(`code`, `message`): [`PluginError`](../../index/classes/PluginError.md)

Defined in: [errors.ts:37](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/errors.ts#L37)

Construct a [PluginError](../../index/classes/PluginError.md) from a Cap'n Proto / JSON wire code.

Unknown codes become `unknown` on [PluginError.code](../../index/classes/PluginError.md#code) while
[PluginError.wireCode](../../index/classes/PluginError.md#wirecode) keeps the raw value.

#### Parameters

##### code

`string`

Wire error code (known or unknown).

##### message

`string`

Operator-facing error text.

#### Returns

[`PluginError`](../../index/classes/PluginError.md)

Typed plugin error.

#### Inherited from

[`PluginError`](../../index/classes/PluginError.md).[`fromWire`](../../index/classes/PluginError.md#fromwire)
