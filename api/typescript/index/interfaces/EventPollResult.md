[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / EventPollResult

# Interface: EventPollResult

Defined in: [generated.ts:2111](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2111)

Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick
off workflows.

## Properties

### users

> **users**: [`ExternalUser`](ExternalUser.md)[]

Defined in: [generated.ts:2113](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2113)

Newly observed external users since the last poll.
