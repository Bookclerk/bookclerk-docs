# bookclerk_plugin_abi::quality_offers

Validation of one `ContentSource.qualityOffers` reply before the host
compares it with the files it holds.

The reply is bytes a jailed storefront guest produced, so the host checks
it here, sans-IO, against the titles it asked about and the stream bounds
below, before any offer can queue an upgrade. The `quality_offers` fuzz
target drives this function (`fuzz/oracles/quality_offers.rs`) with the
same code the host runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`OfferField`](#offerfield) | enum | The field of an offer that broke a bound. |
| [`QualityOffersError`](#qualityofferserror) | enum | Why the host refused a `qualityOffers` reply; no offer from a refused reply is compared. |
| [`validate_quality_offers`](#validate_quality_offers) | function | Checks one `qualityOffers` reply against the titles it answers and the stream bounds. |
| [`decode_quality_offers_bytes`](#decode_quality_offers_bytes) | function | Decodes an unpacked single-segment Cap'n Proto message whose root is a `QualityOffersResult`, with a traversal budget derived from the input size. |
| [`encode_quality_offers_bytes`](#encode_quality_offers_bytes) | function | Encodes `result` as an unpacked single-segment Cap'n Proto message, the inverse of `decode_quality_offers_bytes`. |
| [`MAX_OFFER_CHANNELS`](#max_offer_channels) | constant | Most channels an offered stream may claim, the same bound the host puts on a probed file. |
| [`MAX_OFFER_SAMPLE_RATE_HZ`](#max_offer_sample_rate_hz) | constant | Highest sample rate in Hz an offered stream may claim, the same bound the host puts on a probed file. |
| [`MAX_OFFER_BITRATE_KBPS`](#max_offer_bitrate_kbps) | constant | Highest average bitrate in kbit/s an offered stream may claim: 32 channels of 32-bit PCM at 768 kHz, rounded up, so any real stream fits. |

## Enums

### OfferField

```rust
pub enum OfferField {
    Channels,
    SampleRate,
    Bitrate,
    AacProfile,
}
```

The field of an offer that broke a bound.

**Variants**

- `Channels`: `channels` was above [`MAX_OFFER_CHANNELS`](#max_offer_channels).
- `SampleRate`: `sampleRateHz` was above [`MAX_OFFER_SAMPLE_RATE_HZ`](#max_offer_sample_rate_hz).
- `Bitrate`: `bitrateKbps` was above [`MAX_OFFER_BITRATE_KBPS`](#max_offer_bitrate_kbps).
- `AacProfile`: `aacProfile` was set on a codec that is not AAC.

#### OfferField::as_str

```rust
pub fn as_str(self) -> &'static str
```

The field's wire name, for messages.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### QualityOffersError

```rust
pub enum QualityOffersError {
    TooManyOffers { count: usize, sent: usize },
    Offer { index: usize, field: OfferField },
    RefreshedCredentials(ScanPageError),
}
```

Why the host refused a `qualityOffers` reply; no offer from a refused
reply is compared.

**Variants**

- `TooManyOffers`: The reply answered more titles than the host asked about.
  - `count`: Answers the reply carried.
  - `sent`: Titles the host sent.
- `Offer`: A found offer broke a stream bound.
  - `index`: Index of the offending answer in `offers`.
  - `field`: The field out of bounds.
- `RefreshedCredentials`: `refreshedCredentials` was too large or not one JSON object.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<QualityOffersError>`, `PartialEq`

## Functions

### validate_quality_offers

```rust
pub fn validate_quality_offers(
    result: &QualityOffersResult,
    sent: usize,
) -> Result<(), QualityOffersError>
```

Checks one `qualityOffers` reply against the titles it answers and the
stream bounds.

Only answers marked `found` are checked; the host ignores the offer of
one that is not.

#### Arguments

* `result` - The decoded reply (untrusted).
* `sent` - How many title ids the host sent.

#### Errors

Returns [`QualityOffersError::TooManyOffers`](#qualityofferserror) when the reply answers more
titles than were sent, [`QualityOffersError::Offer`](#qualityofferserror) for the first found
offer out of bounds, and [`QualityOffersError::RefreshedCredentials`](#qualityofferserror) when
the refreshed credentials are too large or not a JSON object.

### decode_quality_offers_bytes

```rust
pub fn decode_quality_offers_bytes(bytes: &[u8]) -> Result<QualityOffersResult>
```

Decodes an unpacked single-segment Cap'n Proto message whose root is a
`QualityOffersResult`, with a traversal budget derived from the input
size.

The host decodes replies through the RPC layer; this byte entry point
serves the `quality_offers` fuzz target and its corpus replay.

#### Arguments

* `bytes` - The message bytes (untrusted).

#### Returns

The decoded reply.

#### Errors

Returns the Cap'n Proto error the framing or the codec reports.

### encode_quality_offers_bytes

```rust
pub fn encode_quality_offers_bytes(result: &QualityOffersResult) -> Result<Vec<u8>>
```

Encodes `result` as an unpacked single-segment Cap'n Proto message, the
inverse of [`decode_quality_offers_bytes`](#decode_quality_offers_bytes).

#### Arguments

* `result` - The reply to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

## Constants

### MAX_OFFER_CHANNELS

```rust
pub const MAX_OFFER_CHANNELS: u32 = 32;
```

Most channels an offered stream may claim, the same bound the host puts
on a probed file.

### MAX_OFFER_SAMPLE_RATE_HZ

```rust
pub const MAX_OFFER_SAMPLE_RATE_HZ: u32 = 768_000;
```

Highest sample rate in Hz an offered stream may claim, the same bound the
host puts on a probed file.

### MAX_OFFER_BITRATE_KBPS

```rust
pub const MAX_OFFER_BITRATE_KBPS: u32 = 800_000;
```

Highest average bitrate in kbit/s an offered stream may claim: 32
channels of 32-bit PCM at 768 kHz, rounded up, so any real stream fits.
