[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / triggerScan

# Function: triggerScan()

> **triggerScan**(): `Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Defined in: [api.ts:2814](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2814)

Queues a full-library scan job for accounts with scan enabled.

Prefer CLI one-shot scans with an explicit `--account` in cloud testing; this
SPA helper hits the authenticated daemon endpoint used by the Library UI.

## Returns

`Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Action ack including `job_id` when accepted.
