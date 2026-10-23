[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / GrantedDatabaseTransport

# Interface: GrantedDatabaseTransport

Defined in: [plugin.ts:547](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L547)

Granted-channel SQL transport the adapter hands to the author isolate for
one named `[[databases]]` binding. The author-side SDK wraps it as the
D1-shaped [DatabaseBinding](DatabaseBinding.md) on `env.<BINDING>`; Python authors call
[GrantedDatabaseTransport.executeBytes](#executebytes) with Cap'n bytes instead.

## Extends

- [`AtomicTransport`](AtomicTransport.md)

## Methods

### execute()

> **execute**(`request`): `Promise`\<[`ExecuteReply`](ExecuteReply.md)\>

Defined in: [db-execute.ts:133](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L133)

Runs one typed atomic batch.

#### Parameters

##### request

[`ExecuteRequest`](ExecuteRequest.md)

Structured Cap'n `ExecuteRequest`.

#### Returns

`Promise`\<[`ExecuteReply`](ExecuteReply.md)\>

Structured `ExecuteReply`.

#### Inherited from

[`AtomicTransport`](AtomicTransport.md).[`execute`](AtomicTransport.md#execute)

***

### executeBytes()

> **executeBytes**(`bytes`): `Promise`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Defined in: [plugin.ts:555](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L555)

Same batch as [AtomicTransport.execute](AtomicTransport.md#execute), as Cap'n Proto bytes
(`ExecuteRequest` in, `ExecuteResultReply` out).

#### Parameters

##### bytes

`Uint8Array`

Unpacked `ExecuteRequest` message.

#### Returns

`Promise`\<`Uint8Array`\<`ArrayBufferLike`\>\>

Unpacked `ExecuteResultReply` message.
