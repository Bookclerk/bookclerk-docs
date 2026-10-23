[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ackEventDelivery

# Function: ackEventDelivery()

> **ackEventDelivery**(`id`): `Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Defined in: [api.ts:2908](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2908)

Acknowledges a dead-lettered delivery (no further retry).

## Parameters

### id

`string`

Delivery id (`{eventId}:{pluginId}`).

## Returns

`Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Action ack from the daemon.
