[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventResult

# Type Alias: EventResult

> **EventResult** = \{ `kind`: `"ack"`; `value`: [`EventAck`](../interfaces/EventAck.md); \} \| \{ `kind`: `"retry"`; `value`: [`EventRetry`](../interfaces/EventRetry.md); \} \| \{ `kind`: `"reject"`; `value`: [`EventReject`](../interfaces/EventReject.md); \} \| \{ `kind`: `"deadLetter"`; `value`: [`EventDeadLetter`](../interfaces/EventDeadLetter.md); \} \| \{ `kind`: `"suspended"`; `value`: [`EventSuspended`](../interfaces/EventSuspended.md); \}

Defined in: [generated.ts:520](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L520)

Per-event outcome inside an `EventConsumer.event` batch reply.
