[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / cancelEventDelivery

# Function: cancelEventDelivery()

> **cancelEventDelivery**(`id`): `Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Defined in: [api.ts:2918](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2918)

Cancels a pending delivery or flags a running one.

## Parameters

### id

`string`

Delivery id (`{eventId}:{pluginId}`).

## Returns

`Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Action ack from the daemon.
