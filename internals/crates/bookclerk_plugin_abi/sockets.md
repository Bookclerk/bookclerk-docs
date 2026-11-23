# bookclerk_plugin_abi::sockets

Sans-IO checks for host-brokered sockets (`Bindings.sockets`).

A socket is a WebSocket (`ws://`, `wss://`) or a raw TLS byte stream
(`tls://host:port`) whose chunks travel as binary frames.

Every side applies the same rules: the guest SDK before it sends, the host
when it serves `Sockets`, and the broker before it dials or answers a
heartbeat. Keeping them here, over plain values, lets the fuzz target and
the unit tests drive the code the broker runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SocketTarget`](#sockettarget) | struct | Where a validated socket URL points. |
| [`SocketScheme`](#socketscheme) | enum | What kind of connection a socket URL asks for. |
| [`socket_frame_len`](#socket_frame_len) | function | Number of bytes a frame carries. |
| [`check_socket_frame`](#check_socket_frame) | function | Fails closed when `frame` is larger than `maxSocketFrameBytes`. |
| [`check_socket_attachment`](#check_socket_attachment) | function | Fails closed when `attachment` is larger than `maxSocketAttachmentBytes`. |
| [`check_socket_id`](#check_socket_id) | function | Checks a socket id: non-empty, at most 64 bytes, `[A-Za-z0-9_-]` only. |
| [`check_socket_close`](#check_socket_close) | function | Normalizes a close request into the code and reason the broker sends. |
| [`parse_socket_url`](#parse_socket_url) | function | Parses a socket URL into its target. |
| [`validate_socket_connect`](#validate_socket_connect) | function | Validates a whole `Sockets.connect` request. |
| [`check_server_name`](#check_server_name) | function | Checks a `tls://` request's server name: empty, or a DNS name of at most `MAX_SOCKET_SERVER_NAME_BYTES` whose dot-separated labels are 1 to 63 letters, digits and inner hyphens. IP literals, punycode (`xn--`) labels and a numeric last label are refused: the name is matched against certificates and grants as text, so it must be one spelling. |
| [`check_trust_anchors`](#check_trust_anchors) | function | Checks a `tls://` request's trust anchors: at most `MAX_SOCKET_TRUST_ANCHORS`, each a DER certificate of at most `MAX_SOCKET_TRUST_ANCHOR_BYTES` whose outer SEQUENCE spans exactly its bytes. The broker's certificate parser decides the rest. |
| [`is_der_certificate_shape`](#is_der_certificate_shape) | function | True when `der` is at most `MAX_SOCKET_TRUST_ANCHOR_BYTES` and is one DER SEQUENCE (tag `0x30`, definite minimal length) covering all of it. |
| [`is_alpn_id`](#is_alpn_id) | function | True when `id` is an ALPN protocol id the broker offers: 1 to `MAX_SOCKET_ALPN_BYTES` bytes of visible ASCII. |
| [`check_socket_frame_for`](#check_socket_frame_for) | function | Checks a frame the guest sends on a socket of `scheme`: a `tls://` stream carries bytes only. |
| [`check_auto_responses`](#check_auto_responses) | function | Checks declared heartbeat pairs: at most `maxSocketAutoResponses`, each frame non-empty and at most `maxSocketAutoResponseBytes`. |
| [`auto_response_for`](#auto_response_for) | function | The reply the broker sends for an incoming frame, when it matches a declared heartbeat exactly (same kind, same bytes). |
| [`MAX_SOCKET_CLOSE_REASON_BYTES`](#max_socket_close_reason_bytes) | constant | Longest close reason a WebSocket close frame can carry, in UTF-8 bytes. |
| [`SOCKET_CLOSE_NORMAL`](#socket_close_normal) | constant | Close code the broker sends when the guest passes zero. |
| [`SOCKET_CLOSE_ABNORMAL`](#socket_close_abnormal) | constant | Close code reported when a connection dropped without a close frame. |
| [`MAX_SOCKET_ALPN_BYTES`](#max_socket_alpn_bytes) | constant | Longest ALPN protocol id a `tls://` stream may offer, in bytes (the TLS limit). |

## Structs

### SocketTarget

```rust
pub struct SocketTarget {
    pub scheme: SocketScheme,
    pub host: String,
    pub port: u16,
}
```

Where a validated socket URL points.

**Fields**

- `scheme`: The kind of connection.
- `host`: Host name or IP literal, lowercased, without brackets.
- `port`: Port, defaulted from the scheme when the URL names none.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### SocketScheme

```rust
pub enum SocketScheme {
    Ws,
    Wss,
    Tls,
}
```

What kind of connection a socket URL asks for.

**Variants**

- `Ws`: `ws://`: a WebSocket over plain TCP.
- `Wss`: `wss://`: a WebSocket over TLS.
- `Tls`: `tls://`: a raw TLS byte stream; frames are binary chunks.

#### SocketScheme::uses_tls

```rust
pub fn uses_tls(self) -> bool
```

True when the connection runs over TLS (`wss://` and `tls://`).

#### SocketScheme::is_stream

```rust
pub fn is_stream(self) -> bool
```

True for a raw byte stream rather than a WebSocket.

#### SocketScheme::as_str

```rust
pub fn as_str(self) -> &'static str
```

The scheme as a URL spells it, lowercase.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### socket_frame_len

```rust
pub fn socket_frame_len(frame: &SocketFrame) -> usize
```

Number of bytes a frame carries.

### check_socket_frame

```rust
pub fn check_socket_frame(frame: &SocketFrame) -> Result<()>
```

Fails closed when `frame` is larger than `maxSocketFrameBytes`.

#### Errors

Returns `payload_too_large` for an oversized frame.

### check_socket_attachment

```rust
pub fn check_socket_attachment(attachment: &[u8]) -> Result<()>
```

Fails closed when `attachment` is larger than `maxSocketAttachmentBytes`.

#### Errors

Returns `payload_too_large` for an oversized attachment.

### check_socket_id

```rust
pub fn check_socket_id(socket: &str) -> Result<()>
```

Checks a socket id: non-empty, at most 64 bytes, `[A-Za-z0-9_-]` only.

#### Errors

Returns `invalid_params` for any other id.

### check_socket_close

```rust
pub fn check_socket_close(code: u16, reason: &str) -> Result<u16>
```

Normalizes a close request into the code and reason the broker sends.

#### Arguments

* `code` - Close code from the guest; zero means 1000.
* `reason` - Close reason from the guest.

#### Returns

The code to send.

#### Errors

Returns `invalid_params` for a code a peer may not send (below 1000, the
reserved 1004 to 1006 and 1015, 1016 to 2999, or above 4999) or a reason
over [`MAX_SOCKET_CLOSE_REASON_BYTES`](#max_socket_close_reason_bytes).

### parse_socket_url

```rust
pub fn parse_socket_url(url: &str) -> Result<SocketTarget>
```

Parses a socket URL into its target.

Only `ws://` and `wss://` URLs with a host, no userinfo and no fragment
are accepted, at most `maxSocketUrlBytes` long, and `tls://host:port`
with an explicit port and nothing after it. A name host may use only
letters, digits, `.`, `-` and `_`, with no punycode (`xn--`) label; a
bracketed host must be an IPv6 address. Percent-encoded hosts fail closed. Whether the host is reachable is the network grant's call, not
this function's.

#### Errors

Returns `invalid_params` naming what is wrong with the URL.

### validate_socket_connect

```rust
pub fn validate_socket_connect(request: &SocketConnectRequest) -> Result<SocketTarget>
```

Validates a whole `Sockets.connect` request.

#### Returns

The parsed target of `request.url`.

#### Errors

Returns `invalid_params` or `payload_too_large` naming the first problem:
the URL, too many or malformed subprotocols or headers, a header the broker
owns, a header value with control bytes, too many or oversized heartbeat
frames, or an oversized attachment.

### check_server_name

```rust
pub fn check_server_name(name: &str) -> Result<()>
```

Checks a `tls://` request's server name: empty, or a DNS name of at most
[`MAX_SOCKET_SERVER_NAME_BYTES`](README.md#max_socket_server_name_bytes) whose dot-separated labels are 1 to 63
letters, digits and inner hyphens. IP literals, punycode (`xn--`)
labels and a numeric last label are refused: the name is matched
against certificates and grants as text, so it must be one spelling.

#### Errors

`invalid_params` naming the problem.

### check_trust_anchors

```rust
pub fn check_trust_anchors(anchors: &[Vec<u8>]) -> Result<()>
```

Checks a `tls://` request's trust anchors: at most
[`MAX_SOCKET_TRUST_ANCHORS`](README.md#max_socket_trust_anchors), each a DER certificate of at most
[`MAX_SOCKET_TRUST_ANCHOR_BYTES`](README.md#max_socket_trust_anchor_bytes) whose outer SEQUENCE spans exactly its
bytes. The broker's certificate parser decides the rest.

#### Errors

`invalid_params` naming the first problem.

### is_der_certificate_shape

```rust
pub fn is_der_certificate_shape(der: &[u8]) -> bool
```

True when `der` is at most [`MAX_SOCKET_TRUST_ANCHOR_BYTES`](README.md#max_socket_trust_anchor_bytes) and is one
DER SEQUENCE (tag `0x30`, definite minimal length) covering all of it.

### is_alpn_id

```rust
pub fn is_alpn_id(id: &str) -> bool
```

True when `id` is an ALPN protocol id the broker offers: 1 to
[`MAX_SOCKET_ALPN_BYTES`](#max_socket_alpn_bytes) bytes of visible ASCII.

### check_socket_frame_for

```rust
pub fn check_socket_frame_for(scheme: SocketScheme, frame: &SocketFrame) -> Result<()>
```

Checks a frame the guest sends on a socket of `scheme`: a `tls://`
stream carries bytes only.

#### Errors

`invalid_params` for a text frame on a stream; the frame size check's
`payload_too_large`.

### check_auto_responses

```rust
pub fn check_auto_responses(pairs: &[SocketAutoResponse]) -> Result<()>
```

Checks declared heartbeat pairs: at most `maxSocketAutoResponses`, each
frame non-empty and at most `maxSocketAutoResponseBytes`.

#### Errors

Returns `invalid_params` naming the first problem.

### auto_response_for

```rust
pub fn auto_response_for<'a>(
    frame: &SocketFrame,
    pairs: &'a [SocketAutoResponse],
) -> Option<&'a SocketFrame>
```

The reply the broker sends for an incoming frame, when it matches a
declared heartbeat exactly (same kind, same bytes).

#### Returns

The declared response of the first matching pair, or `None` when the
frame is for the guest.

## Constants

### MAX_SOCKET_CLOSE_REASON_BYTES

```rust
pub const MAX_SOCKET_CLOSE_REASON_BYTES: usize = 123;
```

Longest close reason a WebSocket close frame can carry, in UTF-8 bytes.

### SOCKET_CLOSE_NORMAL

```rust
pub const SOCKET_CLOSE_NORMAL: u16 = 1000;
```

Close code the broker sends when the guest passes zero.

### SOCKET_CLOSE_ABNORMAL

```rust
pub const SOCKET_CLOSE_ABNORMAL: u16 = 1006;
```

Close code reported when a connection dropped without a close frame.

### MAX_SOCKET_ALPN_BYTES

```rust
pub const MAX_SOCKET_ALPN_BYTES: usize = 255;
```

Longest ALPN protocol id a `tls://` stream may offer, in bytes (the TLS
limit).
