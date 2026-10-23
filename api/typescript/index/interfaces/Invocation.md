[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / Invocation

# Interface: Invocation

Defined in: [generated.ts:2236](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2236)

Identity of one `PluginWorker.open` invocation. The host issues ids; guests
echo `correlationId` / `causationId` onto published events.

## Properties

### accountId

> **accountId**: `string`

Defined in: [generated.ts:2240](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2240)

Account scope; empty for operator / host-wide invocations.

***

### causationId

> **causationId**: `string`

Defined in: [generated.ts:2249](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2249)

Id of the event or command that caused this invocation; empty when none.

***

### correlationId

> **correlationId**: `string`

Defined in: [generated.ts:2247](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2247)

Trace correlation id; empty when none.

***

### deadlineUnixMs

> **deadlineUnixMs**: `number`

Defined in: [generated.ts:2245](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2245)

UTC Unix milliseconds; zero when the invocation has no deadline. The host
fence is authoritative.

***

### id

> **id**: `string`

Defined in: [generated.ts:2238](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2238)

Unique host-issued invocation id.
