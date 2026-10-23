**bookclerk_node_proto > signature**

# Module: signature

## Contents

**Structs**

- [`ArtifactRequest`](#artifactrequest) - What a signed artifact request covers.
- [`AuthContext`](#authcontext) - What a handshake signature covers.

**Enums**

- [`AuthError`](#autherror) - Why a handshake signature was refused.

**Functions**

- [`artifact_transcript`](#artifact_transcript) - The bytes a host signs for an artifact request: the domain tag, then
- [`auth_transcript`](#auth_transcript) - The bytes a host signs: the domain tag, then each field as a 4-byte
- [`canonical_origin`](#canonical_origin) - The ASCII origin (`scheme://host[:port]`) of an http(s) URL, with the
- [`decode_nonce`](#decode_nonce) - Decodes a challenge nonce.
- [`encode_nonce`](#encode_nonce) - Encodes a nonce as base64url without padding.
- [`parse_artifact_authorization`](#parse_artifact_authorization) - Splits an artifact `Authorization` header into node id, issue time and
- [`sign_artifact_request`](#sign_artifact_request) - The `Authorization` header value of a signed artifact request.
- [`sign_auth`](#sign_auth) - Signs a handshake; the result is base64url without padding.
- [`verify_artifact_request`](#verify_artifact_request) - Verifies a signed artifact request with the node's enrolled key and
- [`verify_auth`](#verify_auth) - Verifies a handshake signature with the node's enrolled public key.

**Constants**

- [`ARTIFACT_AUTH_SCHEME`](#artifact_auth_scheme) - `Authorization` scheme of a signed artifact request:
- [`ARTIFACT_DOMAIN`](#artifact_domain) - Domain tag that starts every signed artifact request.
- [`ARTIFACT_REQUEST_MAX_SKEW_SECS`](#artifact_request_max_skew_secs) - How far a signed artifact request's time may be from the owner's clock.
- [`AUTH_DOMAIN`](#auth_domain) - Domain tag that starts every signed transcript.
- [`NONCE_LEN`](#nonce_len) - Length of a challenge nonce.

---

## bookclerk_node_proto::signature::ARTIFACT_AUTH_SCHEME

*Constant*: `&str`

`Authorization` scheme of a signed artifact request:
`Bookclerk-Node <node_id> <issued_at> <sig>`.



## bookclerk_node_proto::signature::ARTIFACT_DOMAIN

*Constant*: `&[u8]`

Domain tag that starts every signed artifact request.



## bookclerk_node_proto::signature::ARTIFACT_REQUEST_MAX_SKEW_SECS

*Constant*: `i64`

How far a signed artifact request's time may be from the owner's clock.



## bookclerk_node_proto::signature::AUTH_DOMAIN

*Constant*: `&[u8]`

Domain tag that starts every signed transcript.



## bookclerk_node_proto::signature::ArtifactRequest

*Struct*

What a signed artifact request covers.

**Generic Parameters:**
- 'a

**Fields:**
- `cluster_id: &'a str` - Owner cluster id.
- `node_id: &'a str` - Requesting node id.
- `owner_origin: &'a str` - Owner origin as [`canonical_origin`](signature.md#bookclerk_node_protosignaturecanonical_origin) spells it.
- `archive_sha256: &'a str` - Lowercase hex SHA-256 of the archive asked for.
- `issued_at: i64` - Unix seconds when the host signed the request.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ArtifactRequest<'a>) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ArtifactRequest<'a>`



## bookclerk_node_proto::signature::AuthContext

*Struct*

What a handshake signature covers.

**Generic Parameters:**
- 'a

**Fields:**
- `cluster_id: &'a str` - Owner cluster id from the challenge.
- `node_id: &'a str` - Node id from the hello.
- `incarnation: &'a str` - Incarnation from the hello.
- `nonce: &'a [u8; 32]` - Nonce from the challenge.
- `owner_origin: &'a str` - Owner origin as [`canonical_origin`](signature.md#bookclerk_node_protosignaturecanonical_origin) spells it. The host uses the

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AuthContext<'a>) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> AuthContext<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::signature::AuthError

*Enum*

Why a handshake signature was refused.

**Variants:**
- `Key` - The stored public key is invalid.
- `Malformed` - The signature is not base64url of 64 bytes.
- `Mismatch` - The signature does not verify.

**Traits:** Error, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AuthError) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`



## bookclerk_node_proto::signature::NONCE_LEN

*Constant*: `usize`

Length of a challenge nonce.



## bookclerk_node_proto::signature::artifact_transcript

*Function*

The bytes a host signs for an artifact request: the domain tag, then
each field as a 4-byte big-endian length and its bytes.

```rust
fn artifact_transcript(request: &ArtifactRequest) -> Vec<u8>
```



## bookclerk_node_proto::signature::auth_transcript

*Function*

The bytes a host signs: the domain tag, then each field as a 4-byte
big-endian length and its bytes.

```rust
fn auth_transcript(ctx: &AuthContext) -> Vec<u8>
```



## bookclerk_node_proto::signature::canonical_origin

*Function*

The ASCII origin (`scheme://host[:port]`) of an http(s) URL, with the
default port and any path dropped and the host lowercased.

```rust
fn canonical_origin(raw: &str) -> Option<String>
```



## bookclerk_node_proto::signature::decode_nonce

*Function*

Decodes a challenge nonce.

```rust
fn decode_nonce(raw: &str) -> Option<[u8; 32]>
```



## bookclerk_node_proto::signature::encode_nonce

*Function*

Encodes a nonce as base64url without padding.

```rust
fn encode_nonce(nonce: &[u8; 32]) -> String
```



## bookclerk_node_proto::signature::parse_artifact_authorization

*Function*

Splits an artifact `Authorization` header into node id, issue time and
signature.

```rust
fn parse_artifact_authorization(header: &str) -> Option<(&str, i64, &str)>
```



## bookclerk_node_proto::signature::sign_artifact_request

*Function*

The `Authorization` header value of a signed artifact request.

```rust
fn sign_artifact_request(key: &ed25519_dalek::SigningKey, request: &ArtifactRequest) -> String
```



## bookclerk_node_proto::signature::sign_auth

*Function*

Signs a handshake; the result is base64url without padding.

```rust
fn sign_auth(key: &ed25519_dalek::SigningKey, ctx: &AuthContext) -> String
```



## bookclerk_node_proto::signature::verify_artifact_request

*Function*

Verifies a signed artifact request with the node's enrolled key and
checks its time against `now` (unix seconds).

# Errors

Returns [`AuthError`](signature.md#bookclerk_node_protosignatureautherror) when the key is invalid, the signature is
malformed or does not verify, or the time is out of range
(`AuthError::Mismatch`).

```rust
fn verify_artifact_request(public_key: &[u8], request: &ArtifactRequest, sig: &str, now: i64) -> Result<(), AuthError>
```



## bookclerk_node_proto::signature::verify_auth

*Function*

Verifies a handshake signature with the node's enrolled public key.
Uses strict verification (no malleable or small-order signatures).

# Errors

Returns [`AuthError`](signature.md#bookclerk_node_protosignatureautherror) when the key is invalid, the signature is
malformed, or it does not verify.

```rust
fn verify_auth(public_key: &[u8], ctx: &AuthContext, sig: &str) -> Result<(), AuthError>
```



