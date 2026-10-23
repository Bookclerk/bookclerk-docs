**bookclerk_plugin_abi > features**

# Module: features

## Contents

**Structs**

- [`RpcFeature`](#rpcfeature) - Known RPC feature names.

**Functions**

- [`negotiate_rpc_features`](#negotiate_rpc_features) - Intersects host-offered and guest-accepted feature lists.

**Constants**

- [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits) - Guest honors scalar / stream-window / list-page caps.
- [`FEATURE_STORAGE_COPY`](#feature_storage_copy) - Guest implements server-side `super::Destination::copy`.
- [`FEATURE_STREAMS`](#feature_streams) - Media moves through transferred [`crate::roles::ByteRange`](roles.md#bookclerk_plugin_abirolesbyterange) / `ByteSource` streams.

---

## bookclerk_plugin_abi::features::FEATURE_SCALAR_LIMITS

*Constant*: `&str`

Guest honors scalar / stream-window / list-page caps.



## bookclerk_plugin_abi::features::FEATURE_STORAGE_COPY

*Constant*: `&str`

Guest implements server-side `super::Destination::copy`.



## bookclerk_plugin_abi::features::FEATURE_STREAMS

*Constant*: `&str`

Media moves through transferred [`crate::roles::ByteRange`](roles.md#bookclerk_plugin_abirolesbyterange) / `ByteSource` streams.



## bookclerk_plugin_abi::features::RpcFeature

*Struct*

Known RPC feature names.

**Fields:**
- `name: &'static str` - Wire name (`rpc.streams`, …).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> RpcFeature`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &RpcFeature) -> bool`



## bookclerk_plugin_abi::features::negotiate_rpc_features

*Function*

Intersects host-offered and guest-accepted feature lists.

# Errors

Returns a message when a required feature is missing or the guest offers an
unsafe combination (streams without scalar limits).

```rust
fn negotiate_rpc_features(host_offers: &[&str], guest_accepts: &[String]) -> crate::Result<Vec<String>>
```



