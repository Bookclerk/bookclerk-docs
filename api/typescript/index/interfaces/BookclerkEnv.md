[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / BookclerkEnv

# Interface: BookclerkEnv

Defined in: [env.ts:273](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L273)

Guest `env` bindings for `api_version = 3` manifests. Every host-provided
facility is a binding here; there is no `HOST` object. Bindings appear only
when `plugin.toml` declares them and the operator has consented.

Run `bookclerk-plugin types` to generate a precise `Env` for one manifest
(`bookclerk-configuration.d.ts`), then `extends BookclerkEntrypoint<Env>`.

## Indexable

> \[`binding`: `string`\]: `unknown`

Named `[[databases]]` bindings, `[[queues.producers]]` bindings, and any
renamed `binding = "..."` entries.

## Properties

### CONFIG?

> `optional` **CONFIG?**: [`JsonObject`](../type-aliases/JsonObject.md)

Defined in: [env.ts:275](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L275)

`[vars]` plus operator settings, decoded from the granted `CONFIG` payload.

***

### EVENTS?

> `optional` **EVENTS?**: [`EventPublisherBinding`](EventPublisherBinding.md)

Defined in: [env.ts:279](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L279)

`[[events.producers]]` outbox publisher.

***

### KV?

> `optional` **KV?**: [`KvNamespaceBinding`](KvNamespaceBinding.md)

Defined in: [env.ts:283](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L283)

`[[kv_namespaces]]` durable KV binding (default name).

***

### OAUTH?

> `optional` **OAUTH?**: `unknown`

Defined in: [env.ts:285](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L285)

`[oauth]` loopback helper; surface reserved.

***

### SECRETS?

> `optional` **SECRETS?**: [`JsonObject`](../type-aliases/JsonObject.md)

Defined in: [env.ts:277](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L277)

`[secrets]` values, decoded from the granted `SECRETS` payload.

***

### WORK\_FS?

> `optional` **WORK\_FS?**: [`StorageBinding`](StorageBinding.md)

Defined in: [env.ts:281](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/env.ts#L281)

`[work_fs]` object storage.
