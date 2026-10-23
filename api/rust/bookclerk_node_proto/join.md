**bookclerk_node_proto > join**

# Module: join

## Contents

**Structs**

- [`OAuthEnrollAnswer`](#oauthenrollanswer) - `POST /api/node/v1/enroll/oauth` answer.
- [`OAuthEnrollRequest`](#oauthenrollrequest) - `POST /api/node/v1/enroll/oauth` body. Keys are base64url without
- [`OwnerInfo`](#ownerinfo) - `GET /api/node/v1/info` answer.

**Functions**

- [`is_branch_name`](#is_branch_name) - True for a branch name: 1 to [`MAX_BRANCH_NAME_BYTES`](join.md#bookclerk_node_protojoinmax_branch_name_bytes) bytes, no control
- [`is_branch_redirect_uri`](#is_branch_redirect_uri) - True for a redirect URI [`BRANCH_CLIENT_ID`](join.md#bookclerk_node_protojoinbranch_client_id) may use: an absolute
- [`is_library_name`](#is_library_name) - True for a library name: 1 to [`MAX_LIBRARY_NAME_BYTES`](join.md#bookclerk_node_protojoinmax_library_name_bytes) bytes, no
- [`pkce_challenge`](#pkce_challenge) - The RFC 7636 S256 challenge of `verifier`: base64url (no padding) of its

**Constants**

- [`AUTHORIZE_PATH`](#authorize_path) - The owner's authorization endpoint (path).
- [`BRANCH_CLIENT_ID`](#branch_client_id) - The built-in public OAuth client a joining branch signs in through. It
- [`BRANCH_CODE_TTL_SECS`](#branch_code_ttl_secs) - Seconds an authorization code for [`BRANCH_CLIENT_ID`](join.md#bookclerk_node_protojoinbranch_client_id) lives.
- [`BRANCH_SCOPE`](#branch_scope) - The only scope [`BRANCH_CLIENT_ID`](join.md#bookclerk_node_protojoinbranch_client_id) may ask for.
- [`ENROLL_OAUTH_PATH`](#enroll_oauth_path) - `POST`: redeem an approved branch sign-in.
- [`INFO_PATH`](#info_path) - `GET`: what a library tells a joining install before sign-in.
- [`JOIN_CALLBACK_PATH`](#join_callback_path) - Path of the joining install's own callback (the SPA route the browser
- [`MAX_BRANCH_NAME_BYTES`](#max_branch_name_bytes) - Longest branch name sent.
- [`MAX_LIBRARY_NAME_BYTES`](#max_library_name_bytes) - Longest library name accepted: a library is named by its public host,

---

## bookclerk_node_proto::join::AUTHORIZE_PATH

*Constant*: `&str`

The owner's authorization endpoint (path).



## bookclerk_node_proto::join::BRANCH_CLIENT_ID

*Constant*: `&str`

The built-in public OAuth client a joining branch signs in through. It
is never stored in the owner's clients table.



## bookclerk_node_proto::join::BRANCH_CODE_TTL_SECS

*Constant*: `i64`

Seconds an authorization code for [`BRANCH_CLIENT_ID`](join.md#bookclerk_node_protojoinbranch_client_id) lives.



## bookclerk_node_proto::join::BRANCH_SCOPE

*Constant*: `&str`

The only scope [`BRANCH_CLIENT_ID`](join.md#bookclerk_node_protojoinbranch_client_id) may ask for.



## bookclerk_node_proto::join::ENROLL_OAUTH_PATH

*Constant*: `&str`

`POST`: redeem an approved branch sign-in.



## bookclerk_node_proto::join::INFO_PATH

*Constant*: `&str`

`GET`: what a library tells a joining install before sign-in.



## bookclerk_node_proto::join::JOIN_CALLBACK_PATH

*Constant*: `&str`

Path of the joining install's own callback (the SPA route the browser
returns to, and the CLI's loopback listener).



## bookclerk_node_proto::join::MAX_BRANCH_NAME_BYTES

*Constant*: `usize`

Longest branch name sent.



## bookclerk_node_proto::join::MAX_LIBRARY_NAME_BYTES

*Constant*: `usize`

Longest library name accepted: a library is named by its public host,
and a DNS name runs to 253 bytes.



## bookclerk_node_proto::join::OAuthEnrollAnswer

*Struct*

`POST /api/node/v1/enroll/oauth` answer.

**Fields:**
- `cluster_id: String` - The library's cluster id.
- `node_id: String` - The node id recorded for the joining install.
- `connection: crate::clerk::ConnectionType` - The connection type recorded (`branch` when left out).
- `bootstrap: Option<String>` - For a clerk: the [`crate::ClerkBootstrap`](clerk.md#bookclerk_node_protoclerkclerkbootstrap) sealed to the node's box

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OAuthEnrollAnswer`
- **PartialEq**
  - `fn eq(self: &Self, other: &OAuthEnrollAnswer) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_node_proto::join::OAuthEnrollRequest

*Struct*

`POST /api/node/v1/enroll/oauth` body. Keys are base64url without
padding.

**Fields:**
- `code: String` - The authorization code from the callback.
- `code_verifier: String` - The PKCE verifier.
- `redirect_uri: String` - The redirect URI the authorization was requested with.
- `sign_pk: String` - Ed25519 public signing key.
- `box_pk: String` - X25519 public box key.
- `branch_name: String` - The name this branch joins as (the node's label).
- `software_version: String` - The joining install's Bookclerk version.
- `connection: crate::clerk::ConnectionType` - The connection type the code was approved for. It must match the

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &OAuthEnrollRequest) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OAuthEnrollRequest`



## bookclerk_node_proto::join::OwnerInfo

*Struct*

`GET /api/node/v1/info` answer.

**Fields:**
- `cluster_id: String` - The library's cluster id.
- `library_name: String` - A name to show for the library.
- `origin: String` - The library's canonical origin: the one the node handshake signs and
- `transport: String` - The node API transport mode (`acme`, `proxy`, `self_signed`,
- `enrollment: bool` - Whether the library lets new branches join (`core.host_enrollment`).
- `spki_fingerprint: Option<String>` - In `self_signed` mode: base64url SHA-256 of the certificate's
- `software_version: String` - The library's Bookclerk version.
- `ancestry: Vec<String>` - The library's ancestry: its own cluster id first, then the cluster

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OwnerInfo`
- **PartialEq**
  - `fn eq(self: &Self, other: &OwnerInfo) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::join::is_branch_name

*Function*

True for a branch name: 1 to [`MAX_BRANCH_NAME_BYTES`](join.md#bookclerk_node_protojoinmax_branch_name_bytes) bytes, no control
characters.

```rust
fn is_branch_name(name: &str) -> bool
```



## bookclerk_node_proto::join::is_branch_redirect_uri

*Function*

True for a redirect URI [`BRANCH_CLIENT_ID`](join.md#bookclerk_node_protojoinbranch_client_id) may use: an absolute
`https` URL, or an `http` URL whose host is a loopback IP literal or
`localhost` (RFC 8252 section 7.3), with no user info and no fragment.

```rust
fn is_branch_redirect_uri(raw: &str) -> bool
```



## bookclerk_node_proto::join::is_library_name

*Function*

True for a library name: 1 to [`MAX_LIBRARY_NAME_BYTES`](join.md#bookclerk_node_protojoinmax_library_name_bytes) bytes, no
control characters.

```rust
fn is_library_name(name: &str) -> bool
```



## bookclerk_node_proto::join::pkce_challenge

*Function*

The RFC 7636 S256 challenge of `verifier`: base64url (no padding) of its
SHA-256.

```rust
fn pkce_challenge(verifier: &str) -> String
```



