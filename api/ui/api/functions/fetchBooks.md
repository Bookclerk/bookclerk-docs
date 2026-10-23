[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchBooks

# Function: fetchBooks()

> **fetchBooks**(`params`): `Promise`\<[`BooksResponse`](../interfaces/BooksResponse.md)\>

Defined in: [api.ts:2789](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2789)

Fetches one page of library books for the Library table.

Omits `status` (or passes `all`) to include every acquire state. Default page
size is 40 when `limit` is omitted.

## Parameters

### params

Optional search string, acquire-status filter, and pagination.

#### limit?

`number`

#### offset?

`number`

#### q?

`string`

#### status?

`string`

## Returns

`Promise`\<[`BooksResponse`](../interfaces/BooksResponse.md)\>

Paginated books plus total count for the pager.
