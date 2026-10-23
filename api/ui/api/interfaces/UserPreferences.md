[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / UserPreferences

# Interface: UserPreferences

Defined in: [api.ts:1576](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1576)

Per-user Discover / default-view preferences.

## Properties

### default\_view

> **default\_view**: [`AppView`](../type-aliases/AppView.md)

Defined in: [api.ts:1577](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1577)

***

### disabled\_shelves

> **disabled\_shelves**: `string`[]

Defined in: [api.ts:1578](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1578)

***

### discover\_excluded\_sources

> **discover\_excluded\_sources**: `string`[]

Defined in: [api.ts:1584](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1584)

Store ids hidden in Discover. Empty = all sources including future.

***

### discover\_language

> **discover\_language**: `string` \| `null`

Defined in: [api.ts:1582](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1582)

`null` / omitted = browser language; `__all__` = no hard filter.

***

### discover\_sort

> **discover\_sort**: [`CatalogSearchSort`](../type-aliases/CatalogSearchSort.md)

Defined in: [api.ts:1579](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1579)

***

### discover\_sort\_dir

> **discover\_sort\_dir**: `"desc"` \| `"asc"`

Defined in: [api.ts:1580](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1580)

***

### theme

> **theme**: `ThemePreference`

Defined in: [api.ts:1586](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1586)

Appearance: `system` follows the OS (fallback light); `light` / `dark` pin it.
