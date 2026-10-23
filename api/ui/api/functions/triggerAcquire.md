[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / triggerAcquire

# Function: triggerAcquire()

> **triggerAcquire**(`body`): `Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Defined in: [api.ts:2830](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2830)

Queues acquire for a library book by uuid and/or ASIN.

## Parameters

### body

Target identifiers (`uuid` and/or `asin`).

#### asin?

`string`

#### uuid?

`string`

## Returns

`Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Action ack including `job_id`.
