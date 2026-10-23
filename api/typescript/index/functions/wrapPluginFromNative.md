[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / wrapPluginFromNative

# Function: wrapPluginFromNative()

> **wrapPluginFromNative**(): *typeof* `InvocationAdapter`

Defined in: [plugin.ts:2906](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2906)

Native-behind-workerd generated adapter: control plane only. The launcher
forwards every entrypoint call to the native guest as typed Cap'n Proto;
this isolate decides `describe` (merged against `PLUGIN_DESCRIBE`) and
`open` policy (`openInvocation`) and receives `shutdown`. There is no
author `PLUGIN` binding.

## Returns

*typeof* `InvocationAdapter`

Adapter entrypoint class bound to [AdapterEnv](../interfaces/AdapterEnv.md).
