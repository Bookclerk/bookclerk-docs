[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / ClusterConsentsResponse

# Interface: ClusterConsentsResponse

Defined in: [api.ts:2426](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2426)

`GET /api/config/cluster-consents`.

## Properties

### clusters

> **clusters**: [`ClusterLinkView`](ClusterLinkView.md)[]

Defined in: [api.ts:2428](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2428)

Every owner cluster link this install holds, in any state.

***

### consents

> **consents**: [`RemoteConsentView`](RemoteConsentView.md)[]

Defined in: [api.ts:2430](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2430)

Every recorded owner plugin request, approved or not.
