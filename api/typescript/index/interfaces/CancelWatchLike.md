[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / CancelWatchLike

# Interface: CancelWatchLike

Defined in: [plugin.ts:1223](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1223)

Host cancellation watch delivered to a job controller (adapter-internal).

## Methods

### wait()

> **wait**(): `Promise`\<`void`\>

Defined in: [plugin.ts:1225](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1225)

Resolves once the host cancels the invocation.

#### Returns

`Promise`\<`void`\>
