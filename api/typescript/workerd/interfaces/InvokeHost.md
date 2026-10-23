[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [workerd](../README.md) / InvokeHost

# Interface: InvokeHost

Defined in: [invoke.ts:128](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L128)

**`Internal`**

Adapter hooks the dispatcher calls back into.

## Methods

### author()

> **author**(): [`AuthorStub`](../../index/type-aliases/AuthorStub.md) \| `undefined`

Defined in: [invoke.ts:130](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L130)

Default author entrypoint (`PLUGIN`), or `undefined` when unbound.

#### Returns

[`AuthorStub`](../../index/type-aliases/AuthorStub.md) \| `undefined`

***

### bindContext()

> **bindContext**(`ctx`): [`GrantedContext`](../../index/interfaces/GrantedContext.md)

Defined in: [invoke.ts:134](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L134)

Bridge context → author-facing granted context.

#### Parameters

##### ctx

`BridgeContext` \| `undefined`

#### Returns

[`GrantedContext`](../../index/interfaces/GrantedContext.md)

***

### importCap()

> **importCap**(`descriptor`, `stop`): `unknown`

Defined in: [invoke.ts:139](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L139)

Resolve one request capability descriptor; `undefined` for an unknown
kind. `stop` aborts when the call returns.

#### Parameters

##### descriptor

[`CapDescriptor`](CapDescriptor.md)

##### stop

`AbortSignal`

#### Returns

`unknown`

***

### named()

> **named**(`name`): [`NamedStub`](../../index/type-aliases/NamedStub.md) \| `undefined`

Defined in: [invoke.ts:132](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L132)

Named author entrypoint, or `undefined` when the isolate does not export it.

#### Parameters

##### name

[`EntrypointName`](../../index/type-aliases/EntrypointName.md)

#### Returns

[`NamedStub`](../../index/type-aliases/NamedStub.md) \| `undefined`
