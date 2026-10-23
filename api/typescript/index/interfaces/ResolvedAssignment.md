[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ResolvedAssignment

# Interface: ResolvedAssignment

Defined in: [generated.ts:2496](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2496)

Destination assignment `lhs = rhs` (INSERT, UPDATE, ...).

## Properties

### column

> **column**: `string`

Defined in: [generated.ts:2500](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2500)

Target column.

***

### dest

> **dest**: `"boolean"` \| `"text"` \| `"integer"` \| `"real"` \| `"blob"` \| `"null"`

Defined in: [generated.ts:2502](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2502)

Declared column type.

***

### source

> **source**: `"boolean"` \| `"text"` \| `"integer"` \| `"real"` \| `"blob"` \| `"null"`

Defined in: [generated.ts:2504](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2504)

Resolved type of the assigned expression.

***

### table

> **table**: `string`

Defined in: [generated.ts:2498](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2498)

Target table.
