# bookclerk_plugin_abi::scan_page

Validation of one `ContentSource.scan` page before the host writes it.

A scan page is bytes a jailed storefront guest produced, so the host
checks it here, sans-IO, against the bounds `schema/plugin.capnp`
declares and against the pass that asked for it, before any title
reaches the library. The `scan_page` fuzz target drives this function
(`fuzz/oracles/scan_page.rs`) with the same code the host runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ScanPageError`](#scanpageerror) | enum | Why the host refused a scan page; nothing from a refused page is written. |
| [`validate_scan_page`](#validate_scan_page) | function | Checks one scan page against the ABI bounds and the pass that asked for it. |
| [`validate_refreshed_credentials`](#validate_refreshed_credentials) | function | Checks credentials a storefront handed back after refreshing them (`ScanPage.refreshedCredentials`, `PlainFetch.refreshedCredentials`) before the host stores them: at most `maxRefreshedCredentialBytes`, and one JSON object, the encoding login credentials are stored in. An empty blob is the wire's "nothing changed" and is accepted; the host stores nothing for it. |
| [`decode_scan_page_bytes`](#decode_scan_page_bytes) | function | Decodes an unpacked single-segment Cap'n Proto message whose root is a `ScanPage`, with a traversal budget derived from the input size. |
| [`encode_scan_page_bytes`](#encode_scan_page_bytes) | function | Encodes `page` as an unpacked single-segment Cap'n Proto message, the inverse of `decode_scan_page_bytes`. |

## Enums

### ScanPageError

```rust
pub enum ScanPageError {
    TooManyBooks { count: usize },
    TooManyRemoved { count: usize },
    CursorTooLong { bytes: usize },
    SyncTokenTooLong { bytes: usize },
    CursorRepeated,
    ForeignAccount { index: usize },
    EmptyProductId { index: usize },
    EmptyRemovedId { index: usize },
    Media { index: usize, error: MediaError },
    RefreshedCredentialsTooLarge { bytes: usize },
    RefreshedCredentialsNotObject,
}
```

Why the host refused a scan page; nothing from a refused page is written.

**Variants**

- `TooManyBooks`: The page carried more titles than `maxScanPageBooks`.
  - `count`: Titles the page carried.
- `TooManyRemoved`: The page carried more removed product ids than `maxScanPageBooks`.
  - `count`: Removed ids the page carried.
- `CursorTooLong`: `nextCursor` was longer than `maxScanCursorBytes`.
  - `bytes`: Length of the cursor in bytes.
- `SyncTokenTooLong`: `syncToken` was longer than `maxScanCursorBytes`.
  - `bytes`: Length of the token in bytes.
- `CursorRepeated`: `nextCursor` repeated the cursor the page answered, so the pass would
  never end.
- `ForeignAccount`: A title named an account other than the one the pass scans.
  - `index`: Index of the offending title in `books`.
- `EmptyProductId`: A title had an empty product id.
  - `index`: Index of the offending title in `books`.
- `EmptyRemovedId`: A removed product id was empty.
  - `index`: Index of the offending id in `removed`.
- `Media`: A title left a kind `unspecified` or named a work kind or carrier
  outside the plugin's `MediaSupport` declaration.
  - `index`: Index of the offending title in `books`.
  - `error`: The rule the title broke.
- `RefreshedCredentialsTooLarge`: `refreshedCredentials` was longer than `maxRefreshedCredentialBytes`.
  - `bytes`: Length of the blob in bytes.
- `RefreshedCredentialsNotObject`: `refreshedCredentials` was not one JSON object, the encoding the host
  stores credentials in.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<ScanPageError>`, `PartialEq`

## Functions

### validate_scan_page

```rust
pub fn validate_scan_page(
    page: &ScanPage,
    sent_cursor: &str,
    account_id: &str,
    media: &MediaSupport,
) -> Result<(), ScanPageError>
```

Checks one scan page against the ABI bounds and the pass that asked for it.

A title whose `account_id` is empty belongs to the pass's account (the
host fills it in); any other value must equal `account_id`. Every title
must name a work kind, expression kind and carrier, and its work kind and
carrier must be inside the plugin's declared `media`.

#### Arguments

* `page` - The page the guest answered (untrusted).
* `sent_cursor` - The cursor the host sent for this page; empty on the
  first page of a pass.
* `account_id` - The account the pass scans.
* `media` - The storefront's `MediaSupport` declaration.

#### Errors

Returns the first [`ScanPageError`](#scanpageerror) the page breaks, checking the counts
and lengths before any per-title field.

### validate_refreshed_credentials

```rust
pub fn validate_refreshed_credentials(bytes: &[u8]) -> Result<(), ScanPageError>
```

Checks credentials a storefront handed back after refreshing them
(`ScanPage.refreshedCredentials`, `PlainFetch.refreshedCredentials`)
before the host stores them: at most `maxRefreshedCredentialBytes`, and
one JSON object, the encoding login credentials are stored in. An empty
blob is the wire's "nothing changed" and is accepted; the host stores
nothing for it.

#### Arguments

* `bytes` - The blob the guest answered (untrusted).

#### Errors

Returns [`ScanPageError::RefreshedCredentialsTooLarge`](#scanpageerror) or
[`ScanPageError::RefreshedCredentialsNotObject`](#scanpageerror).

### decode_scan_page_bytes

```rust
pub fn decode_scan_page_bytes(bytes: &[u8]) -> Result<ScanPage>
```

Decodes an unpacked single-segment Cap'n Proto message whose root is a
`ScanPage`, with a traversal budget derived from the input size.

The host decodes replies through the RPC layer; this byte entry point
serves the `scan_page` fuzz target and its corpus replay.

#### Arguments

* `bytes` - The message bytes (untrusted).

#### Returns

The decoded page.

#### Errors

Returns the Cap'n Proto error the framing or the codec reports.

### encode_scan_page_bytes

```rust
pub fn encode_scan_page_bytes(page: &ScanPage) -> Result<Vec<u8>>
```

Encodes `page` as an unpacked single-segment Cap'n Proto message, the
inverse of [`decode_scan_page_bytes`](#decode_scan_page_bytes).

#### Arguments

* `page` - The page to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.
