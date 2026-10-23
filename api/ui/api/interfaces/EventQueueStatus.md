[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / EventQueueStatus

# Interface: EventQueueStatus

Defined in: [api.ts:210](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L210)

Durable domain-event delivery-queue counters from `/api/status`.

## Properties

### acked

> **acked**: `number`

Defined in: [api.ts:215](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L215)

***

### dead\_letter

> **dead\_letter**: `number`

Defined in: [api.ts:214](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L214)

***

### dead\_letters\_total?

> `optional` **dead\_letters\_total?**: `number`

Defined in: [api.ts:219](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L219)

***

### dispatch\_latency\_ms\_avg?

> `optional` **dispatch\_latency\_ms\_avg?**: `number` \| `null`

Defined in: [api.ts:220](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L220)

***

### handler\_latency\_ms\_avg?

> `optional` **handler\_latency\_ms\_avg?**: `number` \| `null`

Defined in: [api.ts:221](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L221)

***

### oldest\_pending\_age\_secs

> **oldest\_pending\_age\_secs**: `number` \| `null`

Defined in: [api.ts:216](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L216)

***

### pending

> **pending**: `number`

Defined in: [api.ts:211](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L211)

***

### retries\_total?

> `optional` **retries\_total?**: `number`

Defined in: [api.ts:217](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L217)

***

### running

> **running**: `number`

Defined in: [api.ts:212](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L212)

***

### suspended

> **suspended**: `number`

Defined in: [api.ts:213](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L213)

***

### suspensions\_total?

> `optional` **suspensions\_total?**: `number`

Defined in: [api.ts:218](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L218)
