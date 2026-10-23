[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / PluginInstanceConfig

# Interface: PluginInstanceConfig

Defined in: [api.ts:1699](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1699)

Instance document as `GET /api/config/plugin-instances/{id}/config` returns it.

## Properties

### effective

> **effective**: `Record`\<`string`, `unknown`\>

Defined in: [api.ts:1708](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1708)

Schema defaults under the stored settings.

***

### plugin\_instance\_id

> **plugin\_instance\_id**: `string`

Defined in: [api.ts:1700](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1700)

***

### plugin\_key

> **plugin\_key**: `string`

Defined in: [api.ts:1701](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1701)

***

### revision

> **revision**: `number`

Defined in: [api.ts:1702](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1702)

***

### secret\_refs

> **secret\_refs**: `object`[]

Defined in: [api.ts:1706](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1706)

Secret refs (names only).

#### key

> **key**: `string`

#### name

> **name**: `string`

***

### settings

> **settings**: `Record`\<`string`, `string` \| `number` \| `boolean`\>

Defined in: [api.ts:1704](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1704)

Stored settings (operator writes only; defaults are not stored).
