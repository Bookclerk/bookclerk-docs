[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / authMe

# Function: authMe()

> **authMe**(): `Promise`\<[`AuthSession`](../interfaces/AuthSession.md)\>

Defined in: [api.ts:522](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L522)

Loads the current session from `GET /api/auth/me`.

## Returns

`Promise`\<[`AuthSession`](../interfaces/AuthSession.md)\>

Anonymous session when the response is 401.
