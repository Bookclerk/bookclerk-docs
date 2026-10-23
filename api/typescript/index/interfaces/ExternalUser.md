[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ExternalUser

# Interface: ExternalUser

Defined in: [generated.ts:2084](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2084)

One external user observed by an integration. The host may mint claim
tickets without exposing portal details to the guest.

## Properties

### accessToken?

> `optional` **accessToken?**: `string`

Defined in: [generated.ts:2099](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2099)

Ephemeral remote token (e.g. ABS JWT). Guest-to-host only; never
persisted.
Omitted when absent (wire zero value).

***

### displayName?

> `optional` **displayName?**: `string`

Defined in: [generated.ts:2093](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2093)

Display name for UI.
Omitted when absent (wire zero value).

***

### externalUserId

> **externalUserId**: `string`

Defined in: [generated.ts:2088](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2088)

Provider-scoped user id.

***

### provider

> **provider**: `string`

Defined in: [generated.ts:2086](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2086)

Integration provider id (often the plugin id).
