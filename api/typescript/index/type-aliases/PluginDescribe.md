[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PluginDescribe

# Type Alias: PluginDescribe

> **PluginDescribe** = `Partial`\<`WirePluginDescribe`\>

Defined in: [plugin.ts:175](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L175)

Optional identity refinement returned by `BookclerkEntrypoint.describe()`.

The launcher derives the full wire `PluginDescribe` from `plugin.toml`
(`apiVersion`, `id`, `capabilities`, `configOptions`, `cli`, …); an author
`describe()` may add or override presentation fields such as
`displayName`, `brand`, or `rpcFeatures`. Identity, capabilities, and the
`[[settings]]`-derived `configOptions` always come from the manifest: a
guest list that differs from the manifest fails the spawn on the host.
