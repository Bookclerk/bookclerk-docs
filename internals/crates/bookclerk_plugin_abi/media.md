# bookclerk_plugin_abi::media

The media a plugin declares it works with, and the checks the host runs
against that declaration.

Storefronts and metadata providers declare `MediaSupport` in
`plugin.toml` (`[media]`) and in `describe()`. The declaration is
required, never defaulted: a plugin with those entrypoints and an empty
or `unspecified` declaration is refused at spawn, and every record it
returns must name kinds inside it. A `fetchTitle` reply must name a
carrier the plugin declared it fetches and list its files by role. The
checks are sans-IO so the fuzz targets and the host run the same code.

| Item | Kind | Summary |
| --- | --- | --- |
| [`MediaError`](#mediaerror) | enum | Why a `MediaSupport` declaration, or a record against it, was refused. |
| [`FetchError`](#fetcherror) | enum | Why a `fetchTitle` reply (`PlainFetch`) was refused. |
| [`media_support_is_empty`](#media_support_is_empty) | function | Whether a declaration says nothing at all (every list empty). |
| [`validate_media_support`](#validate_media_support) | function | Checks a declaration on its own. |
| [`same_media_support`](#same_media_support) | function | Whether two declarations name the same kinds, ignoring order and repeats: the host requires `describe()` to say what `plugin.toml` says. |
| [`check_record_kinds`](#check_record_kinds) | function | Checks one record's kinds against the plugin's declaration. |
| [`validate_plain_fetch`](#validate_plain_fetch) | function | Checks a `fetchTitle` reply against the plugin's declaration. |
| [`decode_plain_fetch_bytes`](#decode_plain_fetch_bytes) | function | Decodes an unpacked single-segment Cap'n Proto message whose root is a `PlainFetch`, with a traversal budget derived from the input size. |
| [`encode_plain_fetch_bytes`](#encode_plain_fetch_bytes) | function | Encodes `fetch` as an unpacked single-segment Cap'n Proto message, the inverse of `decode_plain_fetch_bytes`. |

## Enums

### MediaError

```rust
pub enum MediaError {
    Missing(&'static str),
    NotAllowed,
    UnspecifiedInDeclaration(&'static str),
    FetchNotListed(MediaKind),
    FetchWithoutStorefront,
    UnspecifiedKind(&'static str),
    WorkKindNotDeclared(WorkKind),
    MediaKindNotDeclared(MediaKind),
}
```

Why a `MediaSupport` declaration, or a record against it, was refused.

**Variants**

- `Missing`: A plugin that must declare its media left `workKinds` or `lists`
  empty.
- `NotAllowed`: A plugin that may not declare media (it exports neither the
  storefront nor the metadata entrypoint) declared some.
- `UnspecifiedInDeclaration`: A declaration list named `unspecified`.
- `FetchNotListed`: `fetches` named a carrier `lists` does not.
- `FetchWithoutStorefront`: A metadata provider declared `fetches`; only storefronts download.
- `UnspecifiedKind`: A record left one of its kinds `unspecified`.
- `WorkKindNotDeclared`: A record's work kind is not in `workKinds`.
- `MediaKindNotDeclared`: A record's carrier is not in `lists`.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### FetchError

```rust
pub enum FetchError {
    UnspecifiedCarrier,
    CarrierNotFetched(MediaKind),
    UnspecifiedRole { index: usize },
    EmptyPath { index: usize },
    NoContent,
    PartsAndWhole,
    SecondWhole,
    SecondCover,
}
```

Why a `fetchTitle` reply ([`PlainFetch`](generated.md#plainfetch)) was refused.

**Variants**

- `UnspecifiedCarrier`: `mediaKind` was left `unspecified`.
- `CarrierNotFetched`: `mediaKind` is not in the plugin's declared `fetches`.
- `UnspecifiedRole`: A file left its role `unspecified`.
  - `index`: Index of the file in `files`.
- `EmptyPath`: A file had an empty path.
  - `index`: Index of the file in `files`.
- `NoContent`: The reply listed neither a `part` nor a `whole` file.
- `PartsAndWhole`: The reply listed both `part`s and a `whole` file.
- `SecondWhole`: The reply listed more than one `whole` file.
- `SecondCover`: The reply listed more than one `cover`.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### media_support_is_empty

```rust
pub fn media_support_is_empty(media: &MediaSupport) -> bool
```

Whether a declaration says nothing at all (every list empty).

### validate_media_support

```rust
pub fn validate_media_support(
    media: &MediaSupport,
    storefront: bool,
    metadata: bool,
) -> Result<(), MediaError>
```

Checks a declaration on its own.

#### Arguments

* `media` - The declaration (from `plugin.toml` or `describe()`).
* `storefront` - Whether the plugin exports the storefront entrypoint.
* `metadata` - Whether the plugin exports the metadata entrypoint.

#### Errors

Returns the first rule the declaration breaks: a storefront or metadata
provider must name at least one work kind and one listed carrier, no
list may name `unspecified`, `fetches` must be a subset of `lists` and
is for storefronts only, and any other plugin must declare nothing.

### same_media_support

```rust
pub fn same_media_support(a: &MediaSupport, b: &MediaSupport) -> bool
```

Whether two declarations name the same kinds, ignoring order and
repeats: the host requires `describe()` to say what `plugin.toml` says.

### check_record_kinds

```rust
pub fn check_record_kinds(
    media: &MediaSupport,
    work: WorkKind,
    expression: ExpressionKind,
    carrier: MediaKind,
) -> Result<(), MediaError>
```

Checks one record's kinds against the plugin's declaration.

#### Arguments

* `media` - The plugin's declaration.
* `work` - The record's work kind.
* `expression` - The record's expression kind.
* `carrier` - The record's media kind.

#### Errors

Returns [`MediaError::UnspecifiedKind`](#mediaerror) when the record leaves a kind
unspecified, or a `*NotDeclared` error when its work kind or carrier is
outside the declaration.

### validate_plain_fetch

```rust
pub fn validate_plain_fetch(
    fetch: &PlainFetch,
    media: &MediaSupport,
) -> Result<(), FetchError>
```

Checks a `fetchTitle` reply against the plugin's declaration.

The carrier must be one the plugin declared in `fetches`, and the files
must describe one edition: ordered `part`s or a single `whole` file, plus
at most one `cover`, each with a role and a path.

#### Arguments

* `fetch` - The guest's reply (untrusted).
* `media` - The plugin's declaration.

#### Errors

Returns the first [`FetchError`](#fetcherror) the reply breaks, checking the carrier
before the files.

### decode_plain_fetch_bytes

```rust
pub fn decode_plain_fetch_bytes(bytes: &[u8]) -> Result<PlainFetch>
```

Decodes an unpacked single-segment Cap'n Proto message whose root is a
`PlainFetch`, with a traversal budget derived from the input size.

The host decodes replies through the RPC layer; this byte entry point
lets the `plain_fetch` fuzz target drive the same reader and validator.

#### Arguments

* `bytes` - The message bytes (untrusted).

#### Returns

The decoded reply.

#### Errors

Returns the Cap'n Proto error the framing or the codec reports.

### encode_plain_fetch_bytes

```rust
pub fn encode_plain_fetch_bytes(fetch: &PlainFetch) -> Result<Vec<u8>>
```

Encodes `fetch` as an unpacked single-segment Cap'n Proto message, the
inverse of [`decode_plain_fetch_bytes`](#decode_plain_fetch_bytes).

#### Arguments

* `fetch` - The reply to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.
