[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [catalogTitle](../README.md) / runtimeBucketBounds

# Function: runtimeBucketBounds()

> **runtimeBucketBounds**(`bucket`): `object`

Defined in: [catalogTitle.ts:172](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/catalogTitle.ts#L172)

Maps a runtime bucket to server min/max length filters.

## Parameters

### bucket

[`RuntimeBucket`](../type-aliases/RuntimeBucket.md)

Selected runtime bucket.

## Returns

`object`

Optional `min_length_minutes` / `max_length_minutes`.

### max\_length\_minutes?

> `optional` **max\_length\_minutes?**: `number`

### min\_length\_minutes?

> `optional` **min\_length\_minutes?**: `number`
