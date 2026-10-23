[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / runSmoke

# Function: runSmoke()

> **runSmoke**(`pluginDir`): `Promise`\<`string`\>

Defined in: [sparse-workerd/smoke.ts:184](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/smoke.ts#L184)

Smokes a workerd plugin directory without the Rust `bookclerk-workerd` binary.

## Parameters

### pluginDir

`string`

Plugin root with `runtime = "workerd"`.

## Returns

`Promise`\<`string`\>

Human-readable success summary including describe/health JSON.

## Throws

When the runtime is not workerd, workerd fails to start, or
  bridge calls fail.

## Example

```ts
console.log(await runSmoke("./my-plugin"));
```
