[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / userAvatarUrl

# Function: userAvatarUrl()

> **userAvatarUrl**(`userId`, `cacheKey?`): `string`

Defined in: [api.ts:1406](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1406)

Authenticated URL for a user's stored profile picture.

## Parameters

### userId

`number`

First-party `users.id`.

### cacheKey?

`number`

Optional cache-buster after upload.

## Returns

`string`

Path used by `<img src>`.
