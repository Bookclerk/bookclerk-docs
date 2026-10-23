[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / AdapterExecuteRequest

# Interface: AdapterExecuteRequest

Defined in: [generated.ts:2630](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2630)

Host -> adapter batch: proven statements plus isolation and receipt.

## Properties

### deadlineUnixMs

> **deadlineUnixMs**: `number`

Defined in: [generated.ts:2638](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2638)

Deadline hint; `0` means none.

***

### isolation

> **isolation**: `"atomicBatch"` \| `"nestedSavepoint"` \| `"consistentSnapshot"`

Defined in: [generated.ts:2640](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2640)

Transaction isolation the adapter must realize.

***

### operationId

> **operationId**: `string`

Defined in: [generated.ts:2632](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2632)

Host-chosen idempotency key.

***

### receipt

> **receipt**: [`AdapterReceipt`](AdapterReceipt.md)

Defined in: [generated.ts:2642](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2642)

Replay receipt to persist; zero/empty when not required.

***

### requestHash

> **requestHash**: `string`

Defined in: [generated.ts:2634](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2634)

SHA-256 hex of the idempotency-relevant request.

***

### statements

> **statements**: [`AdapterStatement`](AdapterStatement.md)[]

Defined in: [generated.ts:2636](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2636)

Statements in execution order.
