**bookclerk_node_proto > sealed**

# Module: sealed

## Contents

**Structs**

- [`SecretEntry`](#secretentry) - One secret in a bundle.
- [`SecretRef`](#secretref) - Names one `encrypted_secrets` row of a store account. It carries no
- [`SecretValue`](#secretvalue) - Secret bytes: zeroized on drop, redacted in `Debug`, and base64url on

**Enums**

- [`SealError`](#sealerror) - Why a sealed box could not be made or opened.

**Functions**

- [`instance_secrets_aad`](#instance_secrets_aad) - Associated data of `instance_id`'s `SECRETS` handed to `node_id` at
- [`open`](#open) - Opens a box sealed to the public key of `recipient_secret`.
- [`open_instance_secrets`](#open_instance_secrets) - Opens a payload made by [`seal_instance_secrets`](sealed.md#bookclerk_node_protosealedseal_instance_secrets) with the node's box
- [`open_secret_update`](#open_secret_update) - Opens a write-back made by [`seal_secret_update`](sealed.md#bookclerk_node_protosealedseal_secret_update) with the owner's
- [`open_secrets`](#open_secrets) - Opens a bundle made by [`seal_secrets`](sealed.md#bookclerk_node_protosealedseal_secrets) with the node's box secret.
- [`open_upload_token`](#open_upload_token) - Opens a token made by [`seal_upload_token`](sealed.md#bookclerk_node_protosealedseal_upload_token) with the node's box secret.
- [`seal`](#seal) - Seals `plaintext` to `recipient` (an X25519 public key) with the
- [`seal_instance_secrets`](#seal_instance_secrets) - Seals an instance's `SECRETS` payload (JSON bytes) for `node_id`'s box
- [`seal_secret_update`](#seal_secret_update) - Seals a refreshed token to the owner's per-connection box key
- [`seal_secrets`](#seal_secrets) - Seals `entries` for `node_id`'s box key `recipient`, bound to `job_id`
- [`seal_upload_token`](#seal_upload_token) - Seals an upload token for `node_id`'s box key `recipient`, bound to the
- [`secret_update_aad`](#secret_update_aad) - Associated data of a token write-back from `node_id` for `secret_ref`,
- [`secrets_aad`](#secrets_aad) - Associated data of the secret bundle handed to `node_id` for `job_id`
- [`upload_token_aad`](#upload_token_aad) - Associated data of the token for upload `upload_id`, granted to

**Constants**

- [`INSTANCE_SECRETS_AAD_DOMAIN`](#instance_secrets_aad_domain) - Domain tag that starts the associated data of an instance's `SECRETS`.
- [`SEALED_BOX_INFO`](#sealed_box_info) - HKDF info string of a sealed box.
- [`SEALED_BOX_OVERHEAD`](#sealed_box_overhead) - Bytes a sealed box adds to its plaintext (ephemeral key and tag).
- [`SECRETS_AAD_DOMAIN`](#secrets_aad_domain) - Domain tag that starts the associated data of a secret bundle.
- [`SECRET_UPDATE_AAD_DOMAIN`](#secret_update_aad_domain) - Domain tag that starts the associated data of a token write-back.
- [`UPLOAD_TOKEN_AAD_DOMAIN`](#upload_token_aad_domain) - Domain tag that starts the associated data of an upload token.

---

## bookclerk_node_proto::sealed::INSTANCE_SECRETS_AAD_DOMAIN

*Constant*: `&[u8]`

Domain tag that starts the associated data of an instance's `SECRETS`.



## bookclerk_node_proto::sealed::SEALED_BOX_INFO

*Constant*: `&[u8]`

HKDF info string of a sealed box.



## bookclerk_node_proto::sealed::SEALED_BOX_OVERHEAD

*Constant*: `usize`

Bytes a sealed box adds to its plaintext (ephemeral key and tag).



## bookclerk_node_proto::sealed::SECRETS_AAD_DOMAIN

*Constant*: `&[u8]`

Domain tag that starts the associated data of a secret bundle.



## bookclerk_node_proto::sealed::SECRET_UPDATE_AAD_DOMAIN

*Constant*: `&[u8]`

Domain tag that starts the associated data of a token write-back.



## bookclerk_node_proto::sealed::SealError

*Enum*

Why a sealed box could not be made or opened.

**Variants:**
- `Key` - The recipient key is not a valid box key.
- `Degenerate` - Key agreement produced the all-zero point.
- `Open` - The box is too short, not base64url, or does not authenticate.
- `Bundle` - The opened bundle is not a JSON list of secrets.

**Traits:** Error, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SealError) -> bool`



## bookclerk_node_proto::sealed::SecretEntry

*Struct*

One secret in a bundle.

**Fields:**
- `secret_ref: SecretRef` - Which row the value belongs to.
- `value: SecretValue` - The unsealed value.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SecretEntry`
- **PartialEq**
  - `fn eq(self: &Self, other: &SecretEntry) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::sealed::SecretRef

*Struct*

Names one `encrypted_secrets` row of a store account. It carries no
secret material.

**Fields:**
- `kind: String` - Secret kind (`source_auth`, `widevine`, ...).
- `provider: String` - Plugin key that owns the secret.
- `account_id: String` - Store account id.
- `name: String` - Logical secret name within that account.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Ord**
  - `fn cmp(self: &Self, other: &SecretRef) -> $crate::cmp::Ordering`
- **Clone**
  - `fn clone(self: &Self) -> SecretRef`
- **PartialEq**
  - `fn eq(self: &Self, other: &SecretRef) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &SecretRef) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_proto::sealed::SecretValue

*Struct*

Secret bytes: zeroized on drop, redacted in `Debug`, and base64url on
the wire.

**Tuple Struct**: `()`

**Methods:**

- `fn new(bytes: Vec<u8>) -> Self` - Wraps secret bytes.
- `fn expose(self: &Self) -> &[u8]` - The secret bytes.
- `fn len(self: &Self) -> usize` - Length in bytes.
- `fn is_empty(self: &Self) -> bool` - True when empty.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<S>(self: &Self, serializer: S) -> Result<<S as >::Ok, <S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SecretValue`
- **PartialEq**
  - `fn eq(self: &Self, other: &SecretValue) -> bool`
- **Deserialize**
  - `fn deserialize<D>(deserializer: D) -> Result<Self, <D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_node_proto::sealed::UPLOAD_TOKEN_AAD_DOMAIN

*Constant*: `&[u8]`

Domain tag that starts the associated data of an upload token.



## bookclerk_node_proto::sealed::instance_secrets_aad

*Function*

Associated data of `instance_id`'s `SECRETS` handed to `node_id` at
document `revision`.

```rust
fn instance_secrets_aad(node_id: &str, instance_id: &str, revision: i64) -> Vec<u8>
```



## bookclerk_node_proto::sealed::open

*Function*

Opens a box sealed to the public key of `recipient_secret`.

# Errors

Returns `SealError::Open` when the box is short, was sealed to another
key or other associated data, or was altered.

```rust
fn open(recipient_secret: &[u8; 32], aad: &[u8], sealed: &[u8]) -> Result<zeroize::Zeroizing<Vec<u8>>, SealError>
```



## bookclerk_node_proto::sealed::open_instance_secrets

*Function*

Opens a payload made by [`seal_instance_secrets`](sealed.md#bookclerk_node_protosealedseal_instance_secrets) with the node's box
secret.

# Errors

Returns `SealError::Open` when it was not sealed to this node for this
instance and revision.

```rust
fn open_instance_secrets(recipient_secret: &[u8; 32], node_id: &str, instance_id: &str, revision: i64, sealed: &str) -> Result<SecretValue, SealError>
```



## bookclerk_node_proto::sealed::open_secret_update

*Function*

Opens a write-back made by [`seal_secret_update`](sealed.md#bookclerk_node_protosealedseal_secret_update) with the owner's
per-connection box secret.

# Errors

Returns `SealError::Open` when it was not sealed to this key for this
node, job, fence and ref.

```rust
fn open_secret_update(owner_box_secret: &[u8; 32], node_id: &str, job_id: &str, fence: i64, secret_ref: &SecretRef, sealed: &str) -> Result<SecretValue, SealError>
```



## bookclerk_node_proto::sealed::open_secrets

*Function*

Opens a bundle made by [`seal_secrets`](sealed.md#bookclerk_node_protosealedseal_secrets) with the node's box secret.

# Errors

Returns `SealError::Open` when the box does not open for this node,
job and fence, and `SealError::Bundle` when its content is malformed.

```rust
fn open_secrets(recipient_secret: &[u8; 32], node_id: &str, job_id: &str, fence: i64, sealed: &str) -> Result<Vec<SecretEntry>, SealError>
```



## bookclerk_node_proto::sealed::open_upload_token

*Function*

Opens a token made by [`seal_upload_token`](sealed.md#bookclerk_node_protosealedseal_upload_token) with the node's box secret.

# Errors

Returns `SealError::Open` when it was not sealed to this node for this
job, fence and upload.

```rust
fn open_upload_token(recipient_secret: &[u8; 32], node_id: &str, job_id: &str, fence: i64, upload_id: &str, sealed: &str) -> Result<SecretValue, SealError>
```



## bookclerk_node_proto::sealed::seal

*Function*

Seals `plaintext` to `recipient` (an X25519 public key) with the
ephemeral secret `ephemeral_secret`, binding `aad`.

`ephemeral_secret` must be fresh random bytes for every box; reusing it
for two plaintexts to one recipient reuses the key and nonce.

# Errors

Returns `SealError::Key` for an invalid recipient key and
`SealError::Degenerate` when key agreement fails.

```rust
fn seal(recipient: &[u8], ephemeral_secret: &[u8; 32], aad: &[u8], plaintext: &[u8]) -> Result<Vec<u8>, SealError>
```



## bookclerk_node_proto::sealed::seal_instance_secrets

*Function*

Seals an instance's `SECRETS` payload (JSON bytes) for `node_id`'s box
key `recipient`, bound to the instance and document revision, as
base64url text for an `instance_config` frame.

# Errors

Returns [`SealError`](sealed.md#bookclerk_node_protosealedsealerror) when the recipient key is invalid.

```rust
fn seal_instance_secrets(recipient: &[u8], ephemeral_secret: &[u8; 32], node_id: &str, instance_id: &str, revision: i64, secrets_json: &[u8]) -> Result<String, SealError>
```



## bookclerk_node_proto::sealed::seal_secret_update

*Function*

Seals a refreshed token to the owner's per-connection box key
`owner_box_key` (from `welcome`), bound to the node, job, fence and
ref, as base64url text for a `secret.update` frame.

# Errors

Returns [`SealError`](sealed.md#bookclerk_node_protosealedsealerror) when the owner key is invalid.

```rust
fn seal_secret_update(owner_box_key: &[u8], ephemeral_secret: &[u8; 32], node_id: &str, job_id: &str, fence: i64, secret_ref: &SecretRef, value: &[u8]) -> Result<String, SealError>
```



## bookclerk_node_proto::sealed::seal_secrets

*Function*

Seals `entries` for `node_id`'s box key `recipient`, bound to `job_id`
and `fence`, as base64url text for a `work` frame.

# Errors

Returns [`SealError`](sealed.md#bookclerk_node_protosealedsealerror) when the recipient key is invalid.

```rust
fn seal_secrets(recipient: &[u8], ephemeral_secret: &[u8; 32], node_id: &str, job_id: &str, fence: i64, entries: &[SecretEntry]) -> Result<String, SealError>
```



## bookclerk_node_proto::sealed::seal_upload_token

*Function*

Seals an upload token for `node_id`'s box key `recipient`, bound to the
job, fence and upload, as base64url text for an `upload.granted` frame.
The token is a bearer credential for the upload endpoint, so it never
travels as frame text.

# Errors

Returns [`SealError`](sealed.md#bookclerk_node_protosealedsealerror) when the recipient key is invalid.

```rust
fn seal_upload_token(recipient: &[u8], ephemeral_secret: &[u8; 32], node_id: &str, job_id: &str, fence: i64, upload_id: &str, token: &[u8]) -> Result<String, SealError>
```



## bookclerk_node_proto::sealed::secret_update_aad

*Function*

Associated data of a token write-back from `node_id` for `secret_ref`,
handed with `job_id` under `fence`.

```rust
fn secret_update_aad(node_id: &str, job_id: &str, fence: i64, secret_ref: &SecretRef) -> Vec<u8>
```



## bookclerk_node_proto::sealed::secrets_aad

*Function*

Associated data of the secret bundle handed to `node_id` for `job_id`
under lease generation `fence`: the domain tag, then each field as a
4-byte big-endian length and its bytes.

```rust
fn secrets_aad(node_id: &str, job_id: &str, fence: i64) -> Vec<u8>
```



## bookclerk_node_proto::sealed::upload_token_aad

*Function*

Associated data of the token for upload `upload_id`, granted to
`node_id` for `job_id` under lease generation `fence`.

```rust
fn upload_token_aad(node_id: &str, job_id: &str, fence: i64, upload_id: &str) -> Vec<u8>
```



