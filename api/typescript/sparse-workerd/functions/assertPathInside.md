[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / assertPathInside

# Function: assertPathInside()

> **assertPathInside**(`root`, `candidate`): `string`

Defined in: [sparse-workerd/ensure.ts:133](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L133)

Resolves `candidate` under `root` via `path.resolve` + `path.relative` barrier.

Rejects NUL bytes and `..` path components (names like `..draft` and
`edition..2` are allowed). Uses the CodeQL-recognized
`path.relative` / `startsWith("..")` containment pattern — no filesystem
probes before the barrier (those are sinks under local threat modeling).

## Parameters

### root

`string`

Trusted directory (resolved).

### candidate

`string`

Absolute path or path relative to `root`.

## Returns

`string`

Absolute resolved path under `root`.

## Throws

When the path is empty, contains NUL/`..`, or escapes `root`.
