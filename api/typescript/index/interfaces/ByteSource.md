[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ByteSource

# Interface: ByteSource

Defined in: [generated.ts:690](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L690)

Transferred readable byte stream. The capability *is* the stream; callers
pull bounded windows. Abort is capability drop / RPC cancel. A failed pull
MUST set `err` — never a successful empty EOF.

## Methods

### pull()

> **pull**(`maxBytes`): `Promise`\<[`PullReply`](../type-aliases/PullReply.md)\>

Defined in: [generated.ts:697](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L697)

Pull the next window of bytes. `done = true` on the final chunk.

#### Parameters

##### maxBytes

`number`

Upper bound for this window; at most `maxStreamWindowBytes`.

#### Returns

`Promise`\<[`PullReply`](../type-aliases/PullReply.md)\>

[PullReply](../type-aliases/PullReply.md)
