[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [workerd](../README.md) / CapDescriptor

# Interface: CapDescriptor

Defined in: [invoke.ts:60](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L60)

**`Internal`**

One `X-Bookclerk-Caps` entry: capability index `i` of the message is
described by entry `i`.

## Properties

### binding?

> `optional` **binding?**: `string`

Defined in: [invoke.ts:66](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L66)

Binding name for `guestDatabase`.

***

### id?

> `optional` **id?**: `string`

Defined in: [invoke.ts:68](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L68)

Isolate object id for `adapterSession`.

***

### kind

> **kind**: `string`

Defined in: [invoke.ts:62](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L62)

`source` / `destination` / `progress` / `cancellation` / … / `adapterSession`.

***

### token?

> `optional` **token?**: `string`

Defined in: [invoke.ts:64](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/invoke.ts#L64)

Grant token for host-served kinds.
