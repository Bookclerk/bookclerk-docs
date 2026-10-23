[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / GrantedFetcher

# Interface: GrantedFetcher

Defined in: [plugin.ts:2177](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2177)

Reverse-channel fetcher bound as `GRANTED` on the adapter isolate.

## Methods

### fetch()

> **fetch**(`input`, `init?`): `Promise`\<`Response`\>

Defined in: [plugin.ts:2185](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L2185)

Call the host grant broker.

#### Parameters

##### input

`string`

Request URL.

##### init?

`RequestInit`

Optional fetch init.

#### Returns

`Promise`\<`Response`\>

Host response.
