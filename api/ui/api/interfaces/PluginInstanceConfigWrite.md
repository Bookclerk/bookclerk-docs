[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginInstanceConfigWrite

# Interface: PluginInstanceConfigWrite

Defined in: [api.ts:1714](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1714)

Body for `PUT /api/config/plugin-instances/{id}/config`.

## Properties

### expected\_revision

> **expected\_revision**: `number`

Defined in: [api.ts:1715](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1715)

***

### secret\_refs

> **secret\_refs**: `object`[]

Defined in: [api.ts:1717](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1717)

#### key

> **key**: `string`

#### name

> **name**: `string`

***

### secrets?

> `optional` **secrets?**: `Record`\<`string`, `string`\>

Defined in: [api.ts:1719](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1719)

Secret values to seal; never echoed back.

***

### settings

> **settings**: `Record`\<`string`, `string` \| `number` \| `boolean`\>

Defined in: [api.ts:1716](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1716)
