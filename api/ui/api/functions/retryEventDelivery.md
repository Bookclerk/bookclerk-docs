[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / retryEventDelivery

# Function: retryEventDelivery()

> **retryEventDelivery**(`id`): `Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Defined in: [api.ts:2898](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2898)

Re-queues a dead-lettered delivery.

## Parameters

### id

`string`

Delivery id (`{eventId}:{pluginId}`).

## Returns

`Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Action ack from the daemon.
