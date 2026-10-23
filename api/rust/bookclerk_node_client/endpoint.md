**bookclerk_node_client > endpoint**

# Module: endpoint

## Contents

**Structs**

- [`OwnerEndpoint`](#ownerendpoint) - Where and how to reach one owner.
- [`TlsProbe`](#tlsprobe) - What a first TLS handshake showed of an owner's certificate, before any

**Enums**

- [`DialError`](#dialerror) - Why the owner could not be reached.

**Traits**

- [`OwnerIo`](#ownerio) - A byte stream to the owner, plain (loopback) or TLS.

**Constants**

- [`CONNECT_PATH`](#connect_path) - Path of the node socket.
- [`ENROLL_PATH`](#enroll_path) - Path of the enroll endpoint.

**Type Aliases**

- [`OwnerSocket`](#ownersocket) - The owner socket.

---

## bookclerk_node_client::endpoint::CONNECT_PATH

*Constant*: `&str`

Path of the node socket.



## bookclerk_node_client::endpoint::DialError

*Enum*

Why the owner could not be reached.

**Variants:**
- `Insecure(&'static str)` - The transport policy refuses this origin (plain HTTP to a
- `Invalid(String)` - The origin or pin is malformed.
- `Io(std::io::Error)` - TCP connect or I/O failed.
- `Tls(String)` - The TLS handshake failed: a wrong pin, an untrusted certificate, or
- `Refused(u16)` - The owner answered the upgrade with an HTTP status.
- `Handshake(String)` - The WebSocket handshake failed otherwise.
- `Http` - The owner's HTTP answer was malformed or too large.
- `Timeout` - A step did not finish in time.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`



## bookclerk_node_client::endpoint::ENROLL_PATH

*Constant*: `&str`

Path of the enroll endpoint.



## bookclerk_node_client::endpoint::OwnerEndpoint

*Struct*

Where and how to reach one owner.

**Methods:**

- `fn new(origin: &str, spki_pin: Option<&str>) -> Result<Self, DialError>` - Checks `origin` (and the link's `spki_pin`) against the transport
- `fn origin(self: &Self) -> &str` - The canonical origin the handshake signs.
- `fn is_pinned(self: &Self) -> bool` - True when the link pins the owner's certificate.
- `fn connect(self: &Self) -> Result<Box<dyn OwnerIo>, DialError>` - Opens a byte stream to the owner under the policy.
- `fn connect_socket(self: &Self) -> Result<OwnerSocket, DialError>` - Dials the node socket, offering [`SUBPROTOCOL`](../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protosubprotocol) and requiring the
- `fn post_json(self: &Self, path: &str, body: &[u8]) -> Result<(u16, Vec<u8>), DialError>` - `POST path` with a JSON body; answers the status and body. The body
- `fn get_json(self: &Self, path: &str, max_body: usize) -> Result<(u16, Vec<u8>), DialError>` - `GET path` of a small JSON answer (at most `max_body` bytes), over the
- `fn get_authorized(self: &Self, path: &str, authorization: &str, max_body: usize) -> Result<(u16, Vec<u8>), DialError>` - `GET path` with an `Authorization` header; answers the status and a
- `fn put_stream<R>(self: &Self, path: &str, authorization: &str, length: u64, body: R) -> Result<(u16, Vec<u8>), DialError>` - `PUT path` streaming exactly `length` bytes from `body`, with an
- `fn post_json_authorized(self: &Self, path: &str, authorization: &str, body: &[u8]) -> Result<(u16, Vec<u8>), DialError>` - `POST path` with a JSON body and an `Authorization` header; answers
- `fn probe_json(self: &Self, path: &str) -> Result<(u16, Vec<u8>, Option<TlsProbe>), DialError>` - `GET path` on an owner whose certificate is not known yet: the TLS

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OwnerEndpoint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_node_client::endpoint::OwnerIo

*Trait*

A byte stream to the owner, plain (loopback) or TLS.



## bookclerk_node_client::endpoint::OwnerSocket

*Type Alias*: `tokio_tungstenite::WebSocketStream<Box<dyn OwnerIo>>`

The owner socket.



## bookclerk_node_client::endpoint::TlsProbe

*Struct*

What a first TLS handshake showed of an owner's certificate, before any
pin is known (`OwnerEndpoint::probe_json`).

**Fields:**
- `spki_pin: String` - base64url SHA-256 of the presented certificate's SubjectPublicKeyInfo.
- `webpki_trusted: bool` - True when the certificate chains to the webpki roots and names the

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &TlsProbe) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> TlsProbe`



