[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Database

# Interface: Database

Defined in: [generated.ts:2776](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2776)

`databaseAdapter` entrypoint (`Entrypoints.databaseAdapter`).

## Methods

### dropUnit()

> **dropUnit**(`unitRef`): `Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

Defined in: [generated.ts:2789](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2789)

Physically drop one provisioned binding unit (`unitRef` is adapter-owned).

#### Parameters

##### unitRef

`string`

Adapter-native unit (sqlite path, postgres database, D1 name).

#### Returns

`Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

[EmptyReply](../type-aliases/EmptyReply.md)

***

### openSession()

> **openSession**(): `Promise`\<[`AdapterSessionReply`](../type-aliases/AdapterSessionReply.md)\>

Defined in: [generated.ts:2782](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2782)

Open one adapter session (capability negotiation + typed execute).

#### Returns

`Promise`\<[`AdapterSessionReply`](../type-aliases/AdapterSessionReply.md)\>

[AdapterSessionReply](../type-aliases/AdapterSessionReply.md)
