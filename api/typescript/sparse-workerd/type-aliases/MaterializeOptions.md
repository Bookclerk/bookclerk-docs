[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / MaterializeOptions

# Type Alias: MaterializeOptions

> **MaterializeOptions** = `object`

Defined in: [sparse-workerd/config.ts:194](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L194)

Options for [materializeConfig](../functions/materializeConfig.md).

## Properties

### bridgeToken

> **bridgeToken**: `string`

Defined in: [sparse-workerd/config.ts:202](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L202)

Per-isolate bearer for `/rpc` and `/health` (`BRIDGE_TOKEN`).

Required — generate once per smoke/isolate and send on every bridge request.

***

### configName?

> `optional` **configName?**: `string`

Defined in: [sparse-workerd/config.ts:215](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L215)

Cap'n Proto output filename under the session dir (default `workerd-config.capnp`).

***

### listenPort

> **listenPort**: `number`

Defined in: [sparse-workerd/config.ts:196](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L196)

Loopback listen port for the HTTP bridge socket (`127.0.0.1:<port>`).

***

### sdkRoot?

> `optional` **sdkRoot?**: `string`

Defined in: [sparse-workerd/config.ts:207](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L207)

Absolute path to the `@bookclerk/plugin-sdk` package root when it differs from
the default (used by unit tests that vendor a fixture package tree).

***

### stateDir?

> `optional` **stateDir?**: `string`

Defined in: [sparse-workerd/config.ts:213](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L213)

Existing session directory for generated embeds (default: allocate under
`$TMPDIR`). Cap'n Proto + `.bookclerk/` are written here — never under the
tainted plugin install root.
