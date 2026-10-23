[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ApiError

# Class: ApiError

Defined in: [api.ts:323](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L323)

HTTP failure from Bookclerk JSON APIs, including optional machine `code`.

## Extends

- `Error`

## Constructors

### Constructor

> **new ApiError**(`status`, `message`, `extras?`): `ApiError`

Defined in: [api.ts:330](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L330)

#### Parameters

##### status

`number`

##### message

`string`

##### extras?

###### code?

`string`

###### pluginId?

`string`

###### summary?

`string`[]

#### Returns

`ApiError`

#### Overrides

`Error.constructor`

## Properties

### code?

> `optional` **code?**: `string`

Defined in: [api.ts:326](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L326)

Machine slug from JSON error bodies when present.

***

### pluginId?

> `optional` **pluginId?**: `string`

Defined in: [api.ts:327](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L327)

***

### status

> **status**: `number`

Defined in: [api.ts:324](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L324)

***

### summary?

> `optional` **summary?**: `string`[]

Defined in: [api.ts:328](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L328)
