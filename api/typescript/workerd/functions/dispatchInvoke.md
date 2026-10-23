[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [workerd](../README.md) / dispatchInvoke

# Function: dispatchInvoke()

> **dispatchInvoke**(`iface`, `method`, `context`, `caps`, `target`, `body`, `host`): `Promise`\<[`InvokeReply`](../interfaces/InvokeReply.md)\>

Defined in: [invoke.ts:757](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L757)

**`Internal`**

Dispatch one `/invoke` call.

## Parameters

### iface

`string`

Cap'n interface name (`X-Bookclerk-Interface`).

### method

`string`

Method name (`X-Bookclerk-Method`).

### context

`BridgeContext` \| `undefined`

Decoded bridge context, or `undefined` for `PluginWorker.*`.

### caps

[`CapDescriptor`](../interfaces/CapDescriptor.md)[]

Request capability descriptors (`X-Bookclerk-Caps`).

### target

`string` \| `null`

Isolate object id (`X-Bookclerk-Target`), or `null`.

### body

`Uint8Array`

`$Params` message bytes.

### host

[`InvokeHost`](../interfaces/InvokeHost.md)

Adapter hooks.

## Returns

`Promise`\<[`InvokeReply`](../interfaces/InvokeReply.md)\>

`$Results` bytes plus reply capability descriptors.

## Throws

On transport-level failures (`400` / `404` / `500`).
