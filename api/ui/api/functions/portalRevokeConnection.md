[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / portalRevokeConnection

# Function: portalRevokeConnection()

> **portalRevokeConnection**(`source`, `accountId`): `Promise`\<`void`\>

Defined in: [api.ts:2729](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2729)

Disconnects a storefront account from the current portal user.

Does not delete library rows; only revokes the sealed credentials / link.
Accounts belong to a plugin, so the source (alias or PluginKey) is part of
the address: the same account id may exist on two stores.

## Parameters

### source

`string`

Source id or PluginKey from [portalConnections](portalConnections.md).

### accountId

`string`

Portal connection account id from [portalConnections](portalConnections.md).

## Returns

`Promise`\<`void`\>

Resolves when the revoke POST succeeds.
