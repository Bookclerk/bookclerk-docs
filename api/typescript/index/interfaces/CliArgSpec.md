[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CliArgSpec

# Interface: CliArgSpec

Defined in: [generated.ts:1242](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1242)

One CLI argument or flag under a `CliCommandSpec`.

## Properties

### about?

> `optional` **about?**: `string`

Defined in: [generated.ts:1268](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1268)

Help text for this arg.
Omitted when absent (wire zero value).

***

### default?

> `optional` **default?**: `string`

Defined in: [generated.ts:1263](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1263)

Default string form when the operator omits the arg.
Omitted when absent (wire zero value).

***

### kind

> **kind**: `"string"` \| `"bool"` \| `"int"` \| `"path"`

Defined in: [generated.ts:1256](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1256)

Parsed value kind.

***

### long?

> `optional` **long?**: `string`

Defined in: [generated.ts:1249](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1249)

Long flag without leading dashes (e.g. "message" -> `--message`).
Omitted when absent (wire zero value).

***

### name

> **name**: `string`

Defined in: [generated.ts:1244](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1244)

Internal arg name used as `CliArg.name` on invoke.

***

### positional

> **positional**: `boolean`

Defined in: [generated.ts:1270](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1270)

When true, the arg is positional rather than a flagged option.

***

### required

> **required**: `boolean`

Defined in: [generated.ts:1258](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1258)

When true, the host rejects invoke if the arg is missing.

***

### short?

> `optional` **short?**: `string`

Defined in: [generated.ts:1254](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1254)

Short flag character (e.g. "m" -> `-m`).
Omitted when absent (wire zero value).
