[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / AdapterStatement

# Interface: AdapterStatement

Defined in: [generated.ts:2614](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2614)

One host-lowered statement with its proof.

## Properties

### kind

> **kind**: `"execute"` \| `"select"` \| `"returning"`

Defined in: [generated.ts:2620](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2620)

Statement classification.

***

### maxRows

> **maxRows**: `number`

Defined in: [generated.ts:2622](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2622)

Row cap for queries; `0` means adapter default.

***

### parameters

> **parameters**: `DbValue`[]

Defined in: [generated.ts:2618](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2618)

Positional bind values.

***

### proof

> **proof**: [`ResolvedStatement`](ResolvedStatement.md)

Defined in: [generated.ts:2626](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2626)

Typed proof bound to `sql`.

***

### resultSelection

> **resultSelection**: `"discard"` \| `"affectedRows"` \| `"rows"`

Defined in: [generated.ts:2624](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2624)

Which outcome parts to return.

***

### sql

> **sql**: `string`

Defined in: [generated.ts:2616](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2616)

Canonical SQL text.
