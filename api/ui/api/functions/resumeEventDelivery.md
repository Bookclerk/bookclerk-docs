[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / resumeEventDelivery

# Function: resumeEventDelivery()

> **resumeEventDelivery**(`id`): `Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Defined in: [api.ts:2928](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2928)

Wakes a suspended delivery for the next worker claim.

## Parameters

### id

`string`

Delivery id (`{eventId}:{pluginId}`).

## Returns

`Promise`\<[`ActionResponse`](../interfaces/ActionResponse.md)\>

Action ack from the daemon.
