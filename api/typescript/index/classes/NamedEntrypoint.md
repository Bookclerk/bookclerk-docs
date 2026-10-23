[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / NamedEntrypoint

# Class: NamedEntrypoint\<Env\>

Defined in: [plugin.ts:1621](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1621)

Base for named entrypoints. Subclasses list their RPC surface on the static
`bookclerkMethods`; the adapter reaches them only through `bookclerkInvoke`,
which installs the granted bindings on `env` before dispatching.

## Extends

- `WorkerEntrypoint`\<`Env`\>

## Extended by

- [`CliEntrypoint`](CliEntrypoint.md)
- [`DatabaseAdapterEntrypoint`](DatabaseAdapterEntrypoint.md)
- [`OidcEntrypoint`](OidcEntrypoint.md)
- [`RemoteLibraryEntrypoint`](RemoteLibraryEntrypoint.md)
- [`StorageEntrypoint`](StorageEntrypoint.md)
- [`StorefrontEntrypoint`](StorefrontEntrypoint.md)

## Type Parameters

### Env

`Env` *extends* [`BookclerkEnv`](../interfaces/BookclerkEnv.md) = [`BookclerkEnv`](../interfaces/BookclerkEnv.md)

## Constructors

### Constructor

> **new NamedEntrypoint**\<`Env`\>(`ctx`, `env`): `NamedEntrypoint`\<`Env`\>

Defined in: [cloudflare-workers.d.ts:15](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L15)

#### Parameters

##### ctx

`ExecutionContext`

##### env

`Env`

#### Returns

`NamedEntrypoint`\<`Env`\>

#### Inherited from

`WorkerEntrypoint<Env>.constructor`

## Properties

### ctx

> `readonly` **ctx**: `ExecutionContext`

Defined in: [cloudflare-workers.d.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L16)

#### Inherited from

`WorkerEntrypoint.ctx`

***

### env

> `readonly` **env**: `Env`

Defined in: [cloudflare-workers.d.ts:17](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L17)

#### Inherited from

`WorkerEntrypoint.env`

***

### invocation

> **invocation**: [`Invocation`](../interfaces/Invocation.md)

Defined in: [plugin.ts:1626](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1626)

Invocation identity of the current call (set by `bookclerkInvoke`).

***

### bookclerkMethods

> `static` **bookclerkMethods**: readonly `string`[] = `[]`

Defined in: [plugin.ts:1623](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1623)

Methods the adapter may dispatch on this entrypoint.

## Methods

### bookclerkInvoke()

> **bookclerkInvoke**(`context`, `method`, ...`args`): `Promise`\<`unknown`\>

Defined in: [plugin.ts:1647](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1647)

**`Internal`**

Adapter dispatch: install granted bindings, then call `method`.

#### Parameters

##### context

[`GrantedContext`](../interfaces/GrantedContext.md)

Granted bindings for this invocation.

##### method

`string`

Method name from the static allowlist.

##### args

...`unknown`[]

Method arguments.

#### Returns

`Promise`\<`unknown`\>

Method result.

***

### fetch()

> **fetch**(`_request?`): `Promise`\<`Response`\>

Defined in: [plugin.ts:1634](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1634)

Rejects HTTP fetch — workerd guests are Workers-RPC only.

#### Parameters

##### \_request?

`Request`

Incoming HTTP request when the entrypoint is fetch-facing.

#### Returns

`Promise`\<`Response`\>

Always a 404 empty response.
