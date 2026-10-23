[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / patchPreferences

# Function: patchPreferences()

> **patchPreferences**(`body`): `Promise`\<[`UserPreferences`](../interfaces/UserPreferences.md)\>

Defined in: [api.ts:1982](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1982)

Partially updates the signed-in user's preferences.

## Parameters

### body

Fields to patch (`discover_language: null` clears to browser default).

#### default_view?

[`AppView`](../type-aliases/AppView.md)

#### disabled_shelves?

`string`[]

#### discover_excluded_sources?

`string`[]

#### discover_language?

`string` \| `null`

Pass `null` to clear to browser default.

#### discover_sort?

[`CatalogSearchSort`](../type-aliases/CatalogSearchSort.md)

#### discover_sort_dir?

`"desc"` \| `"asc"`

#### theme?

`ThemePreference`

## Returns

`Promise`\<[`UserPreferences`](../interfaces/UserPreferences.md)\>

Updated preferences after save.
