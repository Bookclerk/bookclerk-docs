[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / IntegerArithSite

# Interface: IntegerArithSite

Defined in: [generated.ts:2476](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2476)

One INTEGER arithmetic expression that must not wrap or error on overflow.

## Properties

### full

> **full**: [`SqlSpan`](SqlSpan.md)

Defined in: [generated.ts:2478](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2478)

Full expression span (`a + b` or `abs(n)`).

***

### kind

> **kind**: `"add"` \| `"sub"` \| `"mul"` \| `"abs"`

Defined in: [generated.ts:2484](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2484)

Operator.

***

### lhs

> **lhs**: [`SqlSpan`](SqlSpan.md)

Defined in: [generated.ts:2480](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2480)

Left operand (or `abs` argument).

***

### rhs

> **rhs**: [`SqlSpan`](SqlSpan.md)

Defined in: [generated.ts:2482](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2482)

Right operand (`abs` repeats `lhs`).
