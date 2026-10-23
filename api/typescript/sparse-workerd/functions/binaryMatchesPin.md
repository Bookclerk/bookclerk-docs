[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / binaryMatchesPin

# Function: binaryMatchesPin()

> **binaryMatchesPin**(`bin`, `pin`): `boolean`

Defined in: [sparse-workerd/ensure.ts:468](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L468)

True when `bin` is a usable file matching `pin`.

A sibling version stamp is enough when the binary exists. A stamp without a
binary is not current. When no Bookclerk stamp matches, probe `--version`
on the absolute path with fixed argv (`shell: false`, no `PATH` lookup).

## Parameters

### bin

`string`

Absolute candidate executable.

### pin

[`WorkerdPin`](../type-aliases/WorkerdPin.md)

Loaded workerd pin.

## Returns

`boolean`

Whether the binary is present and matches the pin.
