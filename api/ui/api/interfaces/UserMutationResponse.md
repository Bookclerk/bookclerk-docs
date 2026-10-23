[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / UserMutationResponse

# Interface: UserMutationResponse

Defined in: [api.ts:1156](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1156)

Created / patched user response from RBAC provisioning APIs.

## Properties

### claim\_ticket?

> `optional` **claim\_ticket?**: `string` \| `null`

Defined in: [api.ts:1159](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1159)

***

### invite\_url?

> `optional` **invite\_url?**: `string` \| `null`

Defined in: [api.ts:1161](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1161)

Magic-link URL for `/invite?ticket=…` when an invite was minted.

***

### ok

> **ok**: `boolean`

Defined in: [api.ts:1157](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1157)

***

### user

> **user**: [`ListedUser`](ListedUser.md)

Defined in: [api.ts:1158](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L1158)
