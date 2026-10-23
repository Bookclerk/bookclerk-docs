**bookclerk_plugin_abi > jobs**

# Module: jobs

## Contents

**Structs**

- [`StreamCopyHandler`](#streamcopyhandler) - Default [`JobRunner`](roles.md#bookclerk_plugin_abirolesjobrunner) for the stream-copy vertical slice.
- [`StreamCopySpec`](#streamcopyspec) - JSON body for a `stream_copy` [`JobInvocation`](rpc_types.md#bookclerk_plugin_abirpc_typesjobinvocation).

**Functions**

- [`read_all`](#read_all) - Reads a small transferred body, stopping at [`crate::MAX_SCALAR_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes).
- [`stream_copy_keys`](#stream_copy_keys) - Copies `from` → `to` through granted source/destination streams.

---

## bookclerk_plugin_abi::jobs::StreamCopyHandler

*Struct*

Default [`JobRunner`](roles.md#bookclerk_plugin_abirolesjobrunner) for the stream-copy vertical slice.

**Unit Struct**

**Trait Implementations:**

- **JobRunner**
  - `fn job(self: &'life0 Self, controller: JobController) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_abi::jobs::StreamCopySpec

*Struct*

JSON body for a `stream_copy` [`JobInvocation`](rpc_types.md#bookclerk_plugin_abirpc_typesjobinvocation).

**Fields:**
- `from: String` - Source object key.
- `to: String` - Destination object key.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &StreamCopySpec) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> StreamCopySpec`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::jobs::read_all

*Function*

Reads a small transferred body, stopping at [`crate::MAX_SCALAR_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes).

This is not a media reader. A longer body is `PluginError::payload_too_large`
and the extra bytes are not retained.

# Errors

Returns `PluginError::payload_too_large` or an I/O error mapped as
`PluginError::internal`.

```rust
fn read_all(body: std::pin::Pin<Box<dyn tokio::io::AsyncRead>>) -> crate::Result<Vec<u8>>
```



## bookclerk_plugin_abi::jobs::stream_copy_keys

*Function*

Copies `from` → `to` through granted source/destination streams.

Bytes are staged under a retry-stable `commit_token` then committed after a
best-effort fence check. Publication is at-least-once and idempotent on that
token. Cancellation, a failed cancellation poll, or put/commit failure
aborts the stage. A poll error is not treated as "still running".

# Errors

Returns a plugin error when open, put, commit, progress reporting, or
cancellation polling fails.

```rust
fn stream_copy_keys(input: &dyn Source, output: &dyn Destination, spec: &StreamCopySpec, invocation: &crate::rpc_types::JobInvocation, progress: &dyn crate::roles::ProgressSink, cancel: &dyn Cancellation) -> crate::Result<crate::rpc_types::JobOutcome>
```



