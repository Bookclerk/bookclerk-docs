[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventOutcome

# Type Alias: EventOutcome

> **EventOutcome** = \{ `kind`: `"ack"`; \} \| \{ `kind`: `"retry"`; `reason`: `string`; `retryAtUnixMs`: `number`; \} \| \{ `kind`: `"reject"`; `reason`: `string`; \} \| \{ `kind`: `"deadLetter"`; `reason`: `string`; \} \| \{ `checkpointJson`: `string`; `checkpointSchemaVersion`: `number`; `kind`: `"suspended"`; `wakeAtUnixMs`: `number`; `wakeOnEventType`: `string`; `wakeOnFilterJson`: `string`; \}

Defined in: [plugin.ts:785](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L785)

Recorded per-event outcome (`EventResult` union, bridge JSON projection).
