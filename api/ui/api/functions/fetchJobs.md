[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchJobs

# Function: fetchJobs()

> **fetchJobs**(): `Promise`\<[`JobInfo`](../interfaces/JobInfo.md)[]\>

Defined in: [api.ts:2775](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2775)

Lists recent background jobs for the jobs strip / status polling.

## Returns

`Promise`\<[`JobInfo`](../interfaces/JobInfo.md)[]\>

Job rows from `/api/jobs` (newest first as returned by the daemon).
