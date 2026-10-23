**bookclerk_storage > fanout**

# Module: fanout

## Contents

**Structs**

- [`FanoutBackend`](#fanoutbackend) - Multiplexes one logical storage key across multiple backends.

---

## bookclerk_storage::fanout::FanoutBackend

*Struct*

Multiplexes one logical storage key across multiple backends.

Mutations (`put*`, `copy`, `rename`, `delete`, `touch_file`) run on every
child. Reads try children in order and succeed on the first hit.

**Methods:**

- `fn new(backends: Vec<Box<dyn StorageBackend>>) -> Result<Self>` - Build a fan-out over `backends` (must be non-empty).

**Trait Implementations:**

- **StorageBackend**
  - `fn name(self: &Self) -> &'static str`
  - `fn instance_id(self: &Self) -> String`
  - `fn scan_placement(self: &Self) -> Option<String>`
  - `fn clone_box(self: &Self) -> Box<dyn StorageBackend>`
  - `fn put(self: &'life0 Self, key: &'life1 str, data: Bytes, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put_file(self: &'life0 Self, key: &'life1 str, path: &'life2 Path, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn exists(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn probe(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn copy(self: &'life0 Self, from: &'life1 str, to: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn rename(self: &'life0 Self, from: &'life1 str, to: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn delete(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn touch_file(self: &'life0 Self, key: &'life1 str, created: Option<SystemTime>, modified: Option<SystemTime>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_page(self: &'life0 Self, prefix: &'life1 str, cursor: Option<&'life2 str>, limit: u32) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get_stream(self: &'life0 Self, key: &'life1 str, range: Option<crate::ByteRange>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put_stream(self: &'life0 Self, key: &'life1 str, body: std::pin::Pin<Box<dyn tokio::io::AsyncRead>>, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn supports_server_copy(self: &Self) -> bool`



