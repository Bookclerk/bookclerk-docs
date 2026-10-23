**bookclerk_storage > traits**

# Module: traits

## Contents

**Structs**

- [`ByteRange`](#byterange) - Inclusive byte range for a streamed read.
- [`ListPage`](#listpage) - One page of [`ObjectInfo`](traits.md#bookclerk_storagetraitsobjectinfo) from `StorageBackend::list_page`.
- [`ObjectInfo`](#objectinfo) - Listing entry returned by `StorageBackend::list`.
- [`ObjectMeta`](#objectmeta) - Metadata attached to a stored object.
- [`ObjectProbe`](#objectprobe) - Cheap object probe (S3 `HeadObject` / local sidecar meta) — never downloads
- [`PutStreamResult`](#putstreamresult) - Result of `StorageBackend::put_stream`.

**Functions**

- [`bookclerk_meta_sidecar_key`](#bookclerk_meta_sidecar_key) - Sidecar key for probe metadata of this exact object (`{key}.bookclerk-meta.json`).
- [`is_audio_key`](#is_audio_key) - True when `key` ends with a known acquired audio extension.

**Traits**

- [`StorageBackend`](#storagebackend) - Pluggable storage for acquired audio and sidecar files.

**Constants**

- [`AUDIO_EXTENSIONS`](#audio_extensions) - Audio extensions considered acquired media for storage matching.

---

## bookclerk_storage::traits::AUDIO_EXTENSIONS

*Constant*: `&[&str]`

Audio extensions considered acquired media for storage matching.

Includes default remux/encode outputs (`m4b` / `mp3` / `m4a`) plus plain
passthrough containers Chirp / GraphicAudio may store under noop/`as_is`
output (`flac` / `aac` / `ogg` / `oga`). Keep aligned with
`bookclerk_source::media` sniffing and GraphicAudio ZIP audio filters.



## bookclerk_storage::traits::ByteRange

*Struct*

Inclusive byte range for a streamed read.

**Fields:**
- `offset: u64` - Starting offset.
- `length: Option<u64>` - Number of bytes; `None` means to end of object.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ByteRange) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ByteRange`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_storage::traits::ListPage

*Struct*

One page of [`ObjectInfo`](traits.md#bookclerk_storagetraitsobjectinfo) from `StorageBackend::list_page`.

**Fields:**
- `objects: Vec<ObjectInfo>` - Objects in this page.
- `next_cursor: Option<String>` - Continuation token; `None` when this is the last page.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> ListPage`
- **Clone**
  - `fn clone(self: &Self) -> ListPage`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_storage::traits::ObjectInfo

*Struct*

Listing entry returned by `StorageBackend::list`.

**Fields:**
- `key: String` - Relative storage key (prefix + path; no leading slash).
- `size: u64` - Object size in bytes.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ObjectInfo`



## bookclerk_storage::traits::ObjectMeta

*Struct*

Metadata attached to a stored object.

**Fields:**
- `content_type: Option<String>` - MIME type stored with the object (e.g. `audio/mp4`).
- `content_length: Option<u64>` - Object size in bytes when known at put time.
- `asin: Option<String>` - Free-form ASIN / title tags for S3 object metadata.
- `title: Option<String>` - Display title stored as object user-metadata when supported.
- `creation_time: Option<String>` - Creation timestamp as RFC 3339 (S3 metadata `creation-time`).
- `last_write_time: Option<String>` - Last-write timestamp as RFC 3339 (S3 metadata `last-write-time`).
- `sha256_hex: Option<String>` - Lowercase hex SHA-256 (64 chars) of the object body when known.
- `commit_token: Option<String>` - Retry-stable publication token stored with the object, when staged.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> ObjectMeta`
- **Clone**
  - `fn clone(self: &Self) -> ObjectMeta`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_storage::traits::ObjectProbe

*Struct*

Cheap object probe (S3 `HeadObject` / local sidecar meta) — never downloads
object bodies.

**Fields:**
- `key: String` - Relative storage key that was probed.
- `size: u64` - Object size in bytes from HeadObject / local metadata.
- `content_type: Option<String>` - MIME type when the backend exposes it.
- `meta: ObjectMeta` - User-metadata / sidecar fields (ASIN, title, timestamps, …).
- `etag: Option<String>` - Backend entity tag. Opaque; not a content checksum.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> ObjectProbe`
- **Clone**
  - `fn clone(self: &Self) -> ObjectProbe`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_storage::traits::PutStreamResult

*Struct*

Result of `StorageBackend::put_stream`.

**Fields:**
- `bytes_written: u64` - Bytes accepted from the body stream.
- `etag: Option<String>` - Backend etag when available. Opaque; not a content checksum.
- `sha256_hex: Option<String>` - Hex SHA-256 of the bytes accepted, when the backend computed one.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PutStreamResult) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> PutStreamResult`
- **Clone**
  - `fn clone(self: &Self) -> PutStreamResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_storage::traits::StorageBackend

*Trait*

Pluggable storage for acquired audio and sidecar files.

**Methods:**

- `name`: Backend name for logs (`local`, `s3`).
- `instance_id`: Stable identity of this authorized namespace (root, bucket, prefix, session).
- `scan_placement`: Host placement when this backend's bytes are not portable across nodes.
- `clone_box`: Clone into a new boxed backend (same client / root).
- `put`: Write bytes under `key`.
- `put_file`: Stream a local file into storage.
- `get`: Download the full object body into memory.
- `exists`: Return whether `key` exists without downloading the body.
- `list`: Compatibility collector for a namespace that fits in one page.
- `list_audio`: Audio objects under `prefix`, capped at one page of matches.
- `probe`: Probe object metadata without downloading the body.
- `copy`: Copy `from` → `to` within the same backend (S3 server-side copy / local
- `rename`: Move `from` → `to` (copy then delete source).
- `delete`: Delete an object (no-op if missing).
- `head`: Probe metadata without downloading the body.
- `list_page`: One page of keys under `prefix`.
- `get_stream`: Streamed read. Never reassembles the object into host `Bytes`.
- `put_stream`: Streamed write. `body` ownership is transferred to the backend.
- `supports_server_copy`: True when `Self::copy` stays inside this namespace without downloading.
- `touch_file`: Set filesystem timestamps (local) or best-effort logical timestamp tags (S3).



## bookclerk_storage::traits::bookclerk_meta_sidecar_key

*Function*

Sidecar key for probe metadata of this exact object (`{key}.bookclerk-meta.json`).

The record is not shared with another object that only shares a title stem.
`book.m4b` and `book.jpg` keep separate integrity and commit metadata.

```rust
fn bookclerk_meta_sidecar_key(object_key: &str) -> String
```



## bookclerk_storage::traits::is_audio_key

*Function*

True when `key` ends with a known acquired audio extension.

```rust
fn is_audio_key(key: &str) -> bool
```



