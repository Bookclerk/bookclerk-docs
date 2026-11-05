# bookclerk_cover::token

Signed cover tokens: the remote URL of a cover plus a truncated
HMAC-SHA256 under a per-node key, so the cover route only fetches URLs
the API itself handed out.

A token is `<base64url(url)>.<base64url(mac)>`, both unpadded, where `mac`
is the first [`MAC_BYTES`](#mac_bytes) bytes of HMAC-SHA256(key, url). [`open`](#open) is the
parser for tokens arriving in request paths; it is a function over a
string so the `cover_token` fuzz target drives exactly what the daemon
runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`TokenKey`](#tokenkey) | struct | A node's secret for signing cover tokens; never printed. |
| [`is_coverable`](#is_coverable) | function | Whether `url` may get a token: an absolute `https` URL with a host, at most `MAX_URL_BYTES` long. |
| [`sign`](#sign) | function | The token for `url`. |
| [`open`](#open) | function | The URL a token names, when `key` signed it. |
| [`MAX_URL_BYTES`](#max_url_bytes) | constant | Longest cover URL that gets a token, in bytes. |
| [`MAC_BYTES`](#mac_bytes) | constant | Bytes of the HMAC-SHA256 tag a token carries (128 bits). |

## Structs

### TokenKey

```rust
pub struct TokenKey(/* private */);
```

A node's secret for signing cover tokens; never printed.

#### TokenKey::from_bytes

```rust
pub const fn from_bytes(bytes: [u8; 32]) -> Self
```

A key from its 32 bytes.

#### TokenKey::as_bytes

```rust
pub const fn as_bytes(&self) -> &[u8; 32]
```

The key's bytes, for storing it.

Implements: `Clone`, `Debug`

## Functions

### is_coverable

```rust
pub fn is_coverable(url: &str) -> bool
```

Whether `url` may get a token: an absolute `https` URL with a host, at
most [`MAX_URL_BYTES`](#max_url_bytes) long.

### sign

```rust
pub fn sign(key: &TokenKey, url: &str) -> Option<String>
```

The token for `url`.

#### Returns

`None` when `url` is not [coverable](#is_coverable).

### open

```rust
pub fn open(key: &TokenKey, token: &str) -> Option<String>
```

The URL a token names, when `key` signed it.

#### Returns

`None` for a malformed token, a wrong or shortened tag, or a URL that is
not [coverable](#is_coverable).

## Constants

### MAX_URL_BYTES

```rust
pub const MAX_URL_BYTES: usize = 2048;
```

Longest cover URL that gets a token, in bytes.

### MAC_BYTES

```rust
pub const MAC_BYTES: usize = 16;
```

Bytes of the HMAC-SHA256 tag a token carries (128 bits).
