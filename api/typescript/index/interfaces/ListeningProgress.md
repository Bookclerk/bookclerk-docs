[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ListeningProgress

# Interface: ListeningProgress

Defined in: [generated.ts:2125](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2125)

One listening-progress row. The host upserts into `listening_progress`
tagged with the plugin id; plugins never open the library DB.

## Properties

### asin?

> `optional` **asin?**: `string`

Defined in: [generated.ts:2149](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2149)

Amazon ASIN when known.
Omitted when absent (wire zero value).

***

### authors?

> `optional` **authors?**: `string`

Defined in: [generated.ts:2144](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2144)

Authors string when known.
Omitted when absent (wire zero value).

***

### currentTimeSeconds?

> `optional` **currentTimeSeconds?**: `number`

Defined in: [generated.ts:2164](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2164)

Current playback position in seconds.
Omitted when absent (wire zero value).

***

### durationSeconds?

> `optional` **durationSeconds?**: `number`

Defined in: [generated.ts:2169](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2169)

Total duration in seconds when known.
Omitted when absent (wire zero value).

***

### externalItemId

> **externalItemId**: `string`

Defined in: [generated.ts:2129](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2129)

Provider-scoped item / library id.

***

### externalUserId

> **externalUserId**: `string`

Defined in: [generated.ts:2127](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2127)

Provider-scoped user id.

***

### identityId?

> `optional` **identityId?**: `bigint`

Defined in: [generated.ts:2134](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2134)

Bookclerk identity row id when already linked.
Omitted when absent (wire zero value).

***

### isbn?

> `optional` **isbn?**: `string`

Defined in: [generated.ts:2154](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2154)

ISBN when known.
Omitted when absent (wire zero value).

***

### isFinished

> **isFinished**: `boolean`

Defined in: [generated.ts:2171](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2171)

When true, the provider marks the item finished.

***

### lastListenedAtUnixMs?

> `optional` **lastListenedAtUnixMs?**: `number`

Defined in: [generated.ts:2176](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2176)

Last listen timestamp as unix milliseconds (UTC).
Omitted when absent (wire zero value).

***

### progress?

> `optional` **progress?**: `number`

Defined in: [generated.ts:2159](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2159)

Fractional progress in `0.0..=1.0` when the provider reports it.
Omitted when absent (wire zero value).

***

### title?

> `optional` **title?**: `string`

Defined in: [generated.ts:2139](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2139)

Title text when known.
Omitted when absent (wire zero value).
