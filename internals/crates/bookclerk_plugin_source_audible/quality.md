# bookclerk_plugin_source_audible::quality

`qualityOffers`: which stream an account would download for a title
today, read from Audible's content metadata endpoint without licensing
anything.

The request asks for the same DRM path and bitrate tier a download
would (`DownloadOptions`), and the answer's `content_reference` names
the codec and a `<FORMAT>_<kHz>_<kbps>` content format (`AAX_44_128`,
`XHE_44_137`). Spatial codecs (Dolby E-AC-3 and AC-4) are never offered:
Bookclerk never asks Audible for spatial audio, so a title whose
reference names one is answered "not known". So is an xHE-AAC master
when the download would not ask for xHE-AAC.

| Item | Kind | Summary |
| --- | --- | --- |
| [`offer_from_reference`](#offer_from_reference) | function | The offer a title's `content_reference` names, for a download that asks for xHE-AAC when `xhe` is set. |
| [`quality_offers_with_client`](#quality_offers_with_client) | function | Answers `qualityOffers` for every title in `params`, one metadata request each, in order. |

## Functions

### offer_from_reference

```rust
pub fn offer_from_reference(reference: &Value, xhe: bool) -> Option<QualityOffer>
```

The offer a title's `content_reference` names, for a download that asks
for xHE-AAC when `xhe` is set.

#### Returns

`None` when the reference names no codec this guest recognises, a
spatial codec, or xHE-AAC that the download would not ask for.

### quality_offers_with_client

```rust
pub async fn quality_offers_with_client(
    client: &Client,
    marketplace: &str,
    params: &QualityOffersParams,
    options: &DownloadOptions,
) -> Result<QualityOffersResult>
```

Answers `qualityOffers` for every title in `params`, one metadata
request each, in order.

A title whose request fails or whose answer names no usable stream is
answered "not known"; a throttled request stops the batch, since asking
again would only be refused.

#### Arguments

* `client` - A signed-in client for the account.
* `marketplace` - The account's marketplace country code.
* `params` - The titles and the instance's settings.
* `options` - The download options the instance's settings give.

#### Errors

Returns [`AudibleError::Throttled`](README.md#audibleerror) when Audible throttles the account.
