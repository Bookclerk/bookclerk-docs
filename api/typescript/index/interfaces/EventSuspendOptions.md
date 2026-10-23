[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventSuspendOptions

# Interface: EventSuspendOptions

Defined in: [plugin.ts:769](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L769)

Options for [EventMessage.suspend](../classes/EventMessage.md#suspend).

## Properties

### checkpoint?

> `optional` **checkpoint?**: [`Checkpoint`](../type-aliases/Checkpoint.md)

Defined in: [plugin.ts:771](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L771)

Bounded checkpoint replayed on resume (at most `maxCheckpointBytes`).

***

### checkpointSchemaVersion?

> `optional` **checkpointSchemaVersion?**: `number`

Defined in: [plugin.ts:773](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L773)

Schema version of `checkpoint` (default `1`).

***

### wakeAt?

> `optional` **wakeAt?**: [`Instant`](../type-aliases/Instant.md)

Defined in: [plugin.ts:775](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L775)

Earliest resume instant.

***

### wakeOnEventType?

> `optional` **wakeOnEventType?**: `string`

Defined in: [plugin.ts:777](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L777)

Also wake when an event of this type arrives.

***

### wakeOnFilter?

> `optional` **wakeOnFilter?**: `string` \| [`JsonObject`](../type-aliases/JsonObject.md)

Defined in: [plugin.ts:779](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L779)

Host-owned payload filter for wake events (object or JSON text).
