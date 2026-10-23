[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / wrapPluginFromBinding

# Function: wrapPluginFromBinding()

> **wrapPluginFromBinding**(): *typeof* `InvocationAdapter`

Defined in: [plugin.ts:2893](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2893)

Generated adapter isolate: `env.PLUGIN` is the author's default entrypoint
and `env.PLUGIN_<ENTRYPOINT>` the named ones. Every `POST /invoke` call
(Cap'n Proto bytes) and every stream route becomes one `bookclerk*`
dispatch on the author class; authors cannot replace adapter behavior with
adapter-internal names.

## Returns

*typeof* `InvocationAdapter`

Adapter entrypoint class bound to [AdapterEnv](../interfaces/AdapterEnv.md).
