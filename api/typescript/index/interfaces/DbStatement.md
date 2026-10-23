[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DbStatement

# Interface: DbStatement

Defined in: [generated.ts:2436](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2436)

One guest statement in an `ExecuteRequest`.

## Properties

### kind

> **kind**: `"execute"` \| `"select"` \| `"returning"`

Defined in: [generated.ts:2442](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2442)

Statement classification.

***

### maxRows

> **maxRows**: `number`

Defined in: [generated.ts:2444](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2444)

Row cap for queries; `0` means adapter default.

***

### parameters

> **parameters**: `DbValue`[]

Defined in: [generated.ts:2440](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2440)

Positional bind values.

***

### resultSelection

> **resultSelection**: `"discard"` \| `"affectedRows"` \| `"rows"`

Defined in: [generated.ts:2446](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2446)

Which outcome parts to return.

***

### sql

> **sql**: `string`

Defined in: [generated.ts:2438](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2438)

BookclerkSQL text; at most `maxScalarBytes`.
