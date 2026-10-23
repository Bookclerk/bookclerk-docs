[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / GeneratedConfig

# Type Alias: GeneratedConfig

> **GeneratedConfig** = `object`

Defined in: [sparse-workerd/config.ts:221](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L221)

Paths produced by [materializeConfig](../functions/materializeConfig.md).

## Properties

### configPath

> **configPath**: `string`

Defined in: [sparse-workerd/config.ts:223](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L223)

Absolute path to the Cap'n Proto config file (under [stateDir](#statedir)).

***

### importPath

> **importPath**: `string`

Defined in: [sparse-workerd/config.ts:229](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L229)

Pass to `workerd serve --import-path` for `/modules/…` embeds.

***

### listenAddr

> **listenAddr**: `string`

Defined in: [sparse-workerd/config.ts:225](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L225)

Loopback listen address (`127.0.0.1:<port>`).

***

### stateDir

> **stateDir**: `string`

Defined in: [sparse-workerd/config.ts:227](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/config.ts#L227)

Writable session directory holding `.bookclerk/` + Cap'n Proto.
