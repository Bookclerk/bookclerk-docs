[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobInvocation

# Interface: JobInvocation

Defined in: [generated.ts:363](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L363)

Durable command envelope (not a domain event). Command payload schema
version and checkpoint schema versions are independent of the plugin ABI.
The envelope itself is not persisted as opaque bytes across ABI majors.
Idempotency keys are scoped to (account, plugin, commandType) until a
terminal fenced outcome is committed.

## Properties

### attempt

> **attempt**: `number`

Defined in: [generated.ts:375](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L375)

Failure retry counter, starting at 1.

***

### causationId

> **causationId**: `string`

Defined in: [generated.ts:379](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L379)

Id of the event or command that caused this one; empty when none.

***

### checkpointJson

> **checkpointJson**: `string`

Defined in: [generated.ts:386](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L386)

Checkpoint persisted by a prior `SuspendedOutcome`; empty on first run.

***

### checkpointSchemaVersion

> **checkpointSchemaVersion**: `number`

Defined in: [generated.ts:388](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L388)

Schema version of `checkpointJson`.

***

### commandType

> **commandType**: `string`

Defined in: [generated.ts:369](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L369)

Command type the handler dispatches on.

***

### correlationId

> **correlationId**: `string`

Defined in: [generated.ts:377](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L377)

Trace correlation id; empty when none.

***

### deadlineUnixMs

> **deadlineUnixMs**: `number`

Defined in: [generated.ts:384](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L384)

UTC Unix milliseconds. Host fence/lease is authoritative; this hint must
not outlive the fence (clock skew across VPS nodes).

***

### idempotencyKey

> **idempotencyKey**: `string`

Defined in: [generated.ts:373](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L373)

Caller idempotency key scoped to (account, plugin, commandType).

***

### invocationId

> **invocationId**: `string`

Defined in: [generated.ts:367](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L367)

Unique id of this invocation attempt.

***

### invocationSequence

> **invocationSequence**: `number`

Defined in: [generated.ts:390](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L390)

Resume ordinal; distinct from failure `attempt`.

***

### payloadJson

> **payloadJson**: `string`

Defined in: [generated.ts:371](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L371)

Command payload (JSON); at most `maxScalarBytes`.

***

### payloadSchemaVersion

> **payloadSchemaVersion**: `number`

Defined in: [generated.ts:365](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L365)

Schema version of `payloadJson`, owned by the command type.

***

### stepId

> **stepId**: `string`

Defined in: [generated.ts:392](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L392)

Optional step identifier for multi-step commands.
