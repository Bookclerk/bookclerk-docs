[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / loadPin

# Function: loadPin()

> **loadPin**(`root?`): [`WorkerdPin`](../type-aliases/WorkerdPin.md)

Defined in: [sparse-workerd/ensure.ts:57](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L57)

Loads and parses `workerd-pin.json` from the package root.

## Parameters

### root?

`string` = `...`

Package root override (defaults to [packageRoot](packageRoot.md)).

## Returns

[`WorkerdPin`](../type-aliases/WorkerdPin.md)

Parsed pin document.
