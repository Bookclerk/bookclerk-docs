[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / putNodeOwner

# Function: putNodeOwner()

> **putNodeOwner**(`nodeId`, `userId`): `Promise`\<\{ `node_id`: `string`; `owner_user_id`: `number` \| `null`; \}\>

Defined in: [api.ts:2265](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2265)

Assigns a node to the user who administers it, or clears its owner.

## Parameters

### nodeId

`string`

Node to assign.

### userId

`number` \| `null`

Owning user, or `null` to clear.

## Returns

`Promise`\<\{ `node_id`: `string`; `owner_user_id`: `number` \| `null`; \}\>

The node id and its owner.
