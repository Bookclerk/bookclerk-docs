[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DatabaseBindingOptions

# Interface: DatabaseBindingOptions

Defined in: [db-execute.ts:189](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L189)

Options for [createDatabaseBinding](../functions/createDatabaseBinding.md).

## Properties

### deadlineUnixMs?

> `optional` **deadlineUnixMs?**: `number`

Defined in: [db-execute.ts:201](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L201)

Guest-visible deadline (unix ms).

***

### maxRequestBytes?

> `optional` **maxRequestBytes?**: `number`

Defined in: [db-execute.ts:191](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L191)

Negotiated `maxRequestBytes` (`0` = unlimited).

***

### maxResultRows?

> `optional` **maxResultRows?**: `number`

Defined in: [db-execute.ts:193](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L193)

Default `maxRows` for `all()` (`0` = host adapter cap).

***

### operationId?

> `optional` **operationId?**: `string`

Defined in: [db-execute.ts:197](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L197)

Idempotency key; generated when omitted.

***

### requestHash?

> `optional` **requestHash?**: `string`

Defined in: [db-execute.ts:199](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L199)

SHA-256 hex of the request; empty when omitted.

***

### retry?

> `optional` **retry?**: [`RetryToken`](RetryToken.md)

Defined in: [db-execute.ts:195](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L195)

Default retry token; omitted calls mint a UUID and leave the hash empty.
