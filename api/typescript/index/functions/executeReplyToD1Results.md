[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / executeReplyToD1Results

# Function: executeReplyToD1Results()

> **executeReplyToD1Results**(`reply`): [`D1Result`](../interfaces/D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>[]

Defined in: [db-execute.ts:459](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L459)

Maps a typed execute reply to Cloudflare [D1Result](../interfaces/D1Result.md) per statement.

## Parameters

### reply

[`ExecuteReply`](../interfaces/ExecuteReply.md)

Decoded `ExecuteReply` from the host transport.

## Returns

[`D1Result`](../interfaces/D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>[]

One `D1Result` per statement, in plan order.
