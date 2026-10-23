[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / fetchEventDeliveries

# Function: fetchEventDeliveries()

> **fetchEventDeliveries**(`state?`): `Promise`\<[`EventDeliveryInfo`](../interfaces/EventDeliveryInfo.md)[]\>

Defined in: [api.ts:2865](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2865)

Lists recent event deliveries for the Library strip.

## Parameters

### state?

`string`

Optional lifecycle filter (`dead_letter`, `running`, …).

## Returns

`Promise`\<[`EventDeliveryInfo`](../interfaces/EventDeliveryInfo.md)[]\>

Delivery rows from `/api/events/deliveries`.
