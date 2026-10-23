[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / OidcEntrypoint

# Class: OidcEntrypoint\<Env\>

Defined in: [plugin.ts:2120](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2120)

`oidc` entrypoint: relying-party client templates and credential
verification. Export it as `export class Oidc extends OidcEntrypoint`.

## Extends

- [`NamedEntrypoint`](NamedEntrypoint.md)\<`Env`\>

## Type Parameters

### Env

`Env` *extends* [`BookclerkEnv`](../interfaces/BookclerkEnv.md) = [`BookclerkEnv`](../interfaces/BookclerkEnv.md)

## Constructors

### Constructor

> **new OidcEntrypoint**\<`Env`\>(`ctx`, `env`): `OidcEntrypoint`\<`Env`\>

Defined in: [cloudflare-workers.d.ts:15](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L15)

#### Parameters

##### ctx

`ExecutionContext`

##### env

`Env`

#### Returns

`OidcEntrypoint`\<`Env`\>

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`constructor`](NamedEntrypoint.md#constructor)

## Properties

### ctx

> `readonly` **ctx**: `ExecutionContext`

Defined in: [cloudflare-workers.d.ts:16](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L16)

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`ctx`](NamedEntrypoint.md#ctx)

***

### env

> `readonly` **env**: `Env`

Defined in: [cloudflare-workers.d.ts:17](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/cloudflare-workers.d.ts#L17)

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`env`](NamedEntrypoint.md#env-1)

***

### invocation

> **invocation**: [`Invocation`](../interfaces/Invocation.md)

Defined in: [plugin.ts:1626](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1626)

Invocation identity of the current call (set by `bookclerkInvoke`).

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`invocation`](NamedEntrypoint.md#invocation)

***

### bookclerkMethods

> `static` **bookclerkMethods**: readonly `string`[]

Defined in: [plugin.ts:2121](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2121)

Methods the adapter may dispatch on this entrypoint.

#### Overrides

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkMethods`](NamedEntrypoint.md#bookclerkmethods)

## Methods

### authenticateUser()

> **authenticateUser**(`_params`): `Promise`\<[`ExternalUser`](../interfaces/ExternalUser.md)\>

Defined in: [plugin.ts:2138](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2138)

Verifies remote credentials on behalf of the host.

#### Parameters

##### \_params

[`AuthenticateUserParams`](../interfaces/AuthenticateUserParams.md)

Username / password pair.

#### Returns

`Promise`\<[`ExternalUser`](../interfaces/ExternalUser.md)\>

External user identity.

***

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

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`bookclerkInvoke`](NamedEntrypoint.md#bookclerkinvoke)

***

### clients()

> **clients**(): `Promise`\<[`OidcClientTemplate`](../interfaces/OidcClientTemplate.md)[]\>

Defined in: [plugin.ts:2129](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2129)

Plugin-provided OIDC authorization-server client templates. The host
materializes `oidc_clients` rows; plugins never mint tokens.

#### Returns

`Promise`\<[`OidcClientTemplate`](../interfaces/OidcClientTemplate.md)[]\>

Templates (`[]` when unused).

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

#### Inherited from

[`NamedEntrypoint`](NamedEntrypoint.md).[`fetch`](NamedEntrypoint.md#fetch)
