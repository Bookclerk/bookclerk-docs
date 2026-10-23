[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [routes](../README.md) / dropTicketUnlessInvitePath

# Function: dropTicketUnlessInvitePath()

> **dropTicketUnlessInvitePath**(): `void`

Defined in: [routes.ts:164](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/routes.ts#L164)

Invite magic links are `/invite?ticket=` only. Drop `ticket` on any other
path so `/discover?ticket=` cannot open the accept-invite form.

## Returns

`void`
