[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / approveClusterConsent

# Function: approveClusterConsent()

> **approveClusterConsent**(`body`): `Promise`\<\{ `consent`: [`RemoteConsentView`](../interfaces/RemoteConsentView.md); \}\>

Defined in: [api.ts:2491](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2491)

Approves an owner plugin's request, optionally narrowed. Only what the
request asks for can be approved.

## Parameters

### body

[`RemoteApprovalBody`](../interfaces/RemoteApprovalBody.md)

The request reviewed and what to leave out.

## Returns

`Promise`\<\{ `consent`: [`RemoteConsentView`](../interfaces/RemoteConsentView.md); \}\>

The consent row after approval.
