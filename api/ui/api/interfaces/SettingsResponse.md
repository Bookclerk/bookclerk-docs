[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / SettingsResponse

# Interface: SettingsResponse

Defined in: [api.ts:1725](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1725)

Flat config map plus per-plugin setting groups.

## Properties

### effective?

> `optional` **effective?**: `Record`\<`string`, `string`\>

Defined in: [api.ts:1727](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1727)

***

### host\_cpu\_cores\_max?

> `optional` **host\_cpu\_cores\_max?**: `number`

Defined in: [api.ts:1730](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1730)

Host max jail CPU in cores (2 d.p.; equals logical CPU count).

***

### jail\_cpu\_cores?

> `optional` **jail\_cpu\_cores?**: `number` \| `null`

Defined in: [api.ts:1732](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1732)

Optional global per-jail CPU ceiling in cores.

***

### output\_revision?

> `optional` **output\_revision?**: `number`

Defined in: [api.ts:1740](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1740)

`core.output` document revision the `output.*` values were read from; send it
as `expected_output_revision`. `effective["output.revision"]` is the revision
this host has applied, which can lag the document. Absent when the server
could not read `core.output`; the `output.*` values are then missing too and
must not be edited.

***

### plugins

> **plugins**: [`PluginSettingsGroup`](PluginSettingsGroup.md)[]

Defined in: [api.ts:1728](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1728)

***

### settings

> **settings**: `Record`\<`string`, `string`\>

Defined in: [api.ts:1726](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1726)
