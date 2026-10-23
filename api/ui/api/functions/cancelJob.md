[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / cancelJob

# Function: cancelJob()

> **cancelJob**(`id`): `Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Defined in: [api.ts:2849](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2849)

Requests cancel of a pending job or cooperative stop of a running job.

## Parameters

### id

`string`

Durable job id from `/api/jobs`.

## Returns

`Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Action ack including the job id.
