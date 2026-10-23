**bookclerk_storage > local**

# Module: local

## Contents

**Structs**

- [`LocalFsBackend`](#localfsbackend) - Stores objects under a root directory; keys map to relative paths.

---

## bookclerk_storage::local::LocalFsBackend

*Struct*

Stores objects under a root directory; keys map to relative paths.

An optional key prefix is prepended to every key (same model as S3),
so library `storage_key` values stay relative to the prefix.

**Methods:**

- `fn new(root: PathBuf) -> Result<Self>` - Create a backend rooted at `root` with no key prefix.
- `fn with_prefix(root: PathBuf, prefix: &str) -> Result<Self>` - Create a backend rooted at `root` with an optional key prefix
- `fn with_scan_placement<impl Into<String>>(self: Self, placement: impl Trait) -> Self` - Overrides the host placement reported for scan adoption.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LocalFsBackend`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **StorageBackend**
  - `fn name(self: &Self) -> &'static str`
  - `fn instance_id(self: &Self) -> String`
  - `fn scan_placement(self: &Self) -> Option<String>`
  - `fn supports_server_copy(self: &Self) -> bool`
  - `fn clone_box(self: &Self) -> Box<dyn StorageBackend>`
  - `fn put(self: &'life0 Self, key: &'life1 str, data: Bytes, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put_file(self: &'life0 Self, key: &'life1 str, source: &'life2 Path, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn exists(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn probe(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn copy(self: &'life0 Self, from: &'life1 str, to: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn delete(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn touch_file(self: &'life0 Self, key: &'life1 str, created: Option<SystemTime>, modified: Option<SystemTime>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list_page(self: &'life0 Self, prefix: &'life1 str, cursor: Option<&'life2 str>, limit: u32) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get_stream(self: &'life0 Self, key: &'life1 str, range: Option<crate::ByteRange>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put_stream(self: &'life0 Self, key: &'life1 str, body: std::pin::Pin<Box<dyn tokio::io::AsyncRead>>, meta: ObjectMeta) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



