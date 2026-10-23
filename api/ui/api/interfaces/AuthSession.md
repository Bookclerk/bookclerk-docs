[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / AuthSession

# Interface: AuthSession

Defined in: [api.ts:79](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L79)

Normalized client session used to gate SPA views and actions.

## Properties

### authenticated

> **authenticated**: `boolean`

Defined in: [api.ts:80](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L80)

***

### can\_acquire

> **can\_acquire**: `boolean`

Defined in: [api.ts:83](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L83)

***

### default\_view

> **default\_view**: [`AppView`](../type-aliases/AppView.md)

Defined in: [api.ts:82](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L82)

***

### elevated?

> `optional` **elevated?**: `boolean`

Defined in: [api.ts:84](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L84)

***

### impersonating?

> `optional` **impersonating?**: [`AuthMeImpersonating`](AuthMeImpersonating.md)

Defined in: [api.ts:85](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L85)

***

### portal?

> `optional` **portal?**: [`PortalInfo`](PortalInfo.md)

Defined in: [api.ts:86](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L86)

***

### role?

> `optional` **role?**: [`AuthRole`](../type-aliases/AuthRole.md)

Defined in: [api.ts:81](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L81)

***

### second\_factor?

> `optional` **second\_factor?**: [`AuthSecondFactor`](AuthSecondFactor.md)

Defined in: [api.ts:88](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L88)

***

### user?

> `optional` **user?**: [`AuthMeUser`](AuthMeUser.md)

Defined in: [api.ts:87](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L87)
