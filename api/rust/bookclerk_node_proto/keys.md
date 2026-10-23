**bookclerk_node_proto > keys**

# Module: keys

## Contents

**Enums**

- [`KeyError`](#keyerror) - Why a public key was refused.

**Functions**

- [`box_public_key`](#box_public_key) - The X25519 public key of a 32-byte secret (clamped per RFC 7748).
- [`sign_public_key`](#sign_public_key) - The Ed25519 public key of a 32-byte signing seed.
- [`validate_box_public_key`](#validate_box_public_key) - Validates an X25519 public box key (a Montgomery u-coordinate).
- [`validate_sign_public_key`](#validate_sign_public_key) - Validates an Ed25519 public signing key.

**Constants**

- [`PUBLIC_KEY_LEN`](#public_key_len) - Length of either public key.

---

## bookclerk_node_proto::keys::KeyError

*Enum*

Why a public key was refused.

**Variants:**
- `Length` - Not 32 bytes.
- `NotOnCurve` - Not a point on the curve.
- `NonCanonical` - A non-canonical encoding of a point.
- `WeakPoint` - A small-order or mixed-order point.

**Traits:** Eq, Error

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &KeyError) -> bool`



## bookclerk_node_proto::keys::PUBLIC_KEY_LEN

*Constant*: `usize`

Length of either public key.



## bookclerk_node_proto::keys::box_public_key

*Function*

The X25519 public key of a 32-byte secret (clamped per RFC 7748).

```rust
fn box_public_key(secret: &[u8; 32]) -> [u8; 32]
```



## bookclerk_node_proto::keys::sign_public_key

*Function*

The Ed25519 public key of a 32-byte signing seed.

```rust
fn sign_public_key(seed: &[u8; 32]) -> [u8; 32]
```



## bookclerk_node_proto::keys::validate_box_public_key

*Function*

Validates an X25519 public box key (a Montgomery u-coordinate).

# Errors

Returns [`KeyError`](keys.md#bookclerk_node_protokeyskeyerror) when the key is the wrong length, on the twist,
not canonical, or not in the prime-order subgroup.

```rust
fn validate_box_public_key(raw: &[u8]) -> Result<(), KeyError>
```



## bookclerk_node_proto::keys::validate_sign_public_key

*Function*

Validates an Ed25519 public signing key.

# Errors

Returns [`KeyError`](keys.md#bookclerk_node_protokeyskeyerror) when the key is the wrong length, not a canonical
curve point, or not in the prime-order subgroup.

```rust
fn validate_sign_public_key(raw: &[u8]) -> Result<ed25519_dalek::VerifyingKey, KeyError>
```



