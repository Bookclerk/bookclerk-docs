**bookclerk_storage > bounded**

# Module: bounded

## Contents

**Enums**

- [`ReadSpan`](#readspan) - How a ranged read is applied after rejecting ambiguous internal lengths.

**Functions**

- [`ensure_scalar_len`](#ensure_scalar_len) - Rejects a scalar put whose buffer already exceeds `limit`.
- [`normalize_range`](#normalize_range) - Normalizes an internal range.
- [`parse_sha256_hex`](#parse_sha256_hex) - Parses a hex SHA-256 (64 hex chars) into raw bytes.
- [`read_scalar_body`](#read_scalar_body) - Reads at most `limit + 1` bytes.
- [`reject_scalar_hint`](#reject_scalar_hint) - Early-reject when a HEAD/stat hint already exceeds `limit`.
- [`sha256_field_from_raw`](#sha256_field_from_raw) - Validates a raw SHA-256 field (ABI `Data`, 32 bytes) into hex.

**Constants**

- [`MAX_LIST_PAGE`](#max_list_page) - ABI list page ceiling.
- [`MAX_SCALAR_OBJECT_BYTES`](#max_scalar_object_bytes) - ABI scalar ceiling. Audiobook bodies must not use `crate::StorageBackend::get` /
- [`MAX_SUPPORTED_OBJECT_BYTES`](#max_supported_object_bytes) - Largest object this process will upload or server-copy.

---

## bookclerk_storage::bounded::MAX_LIST_PAGE

*Constant*: `u32`

ABI list page ceiling.



## bookclerk_storage::bounded::MAX_SCALAR_OBJECT_BYTES

*Constant*: `u64`

ABI scalar ceiling. Audiobook bodies must not use `crate::StorageBackend::get` /
`crate::StorageBackend::put`.



## bookclerk_storage::bounded::MAX_SUPPORTED_OBJECT_BYTES

*Constant*: `u64`

Largest object this process will upload or server-copy.

S3 allows 10,000 parts. Each part is 8 MiB, so the application ceiling is
80 GiB. AWS's own 5 TiB object maximum is wider;
requests above this ceiling fail before any part buffer is filled.



## bookclerk_storage::bounded::ReadSpan

*Enum*

How a ranged read is applied after rejecting ambiguous internal lengths.

**Variants:**
- `ToEnd{ offset: u64 }` - Bytes from `offset` through EOF.
- `Exact{ offset: u64, length: u64 }` - Exactly `length` bytes starting at `offset`.



## bookclerk_storage::bounded::ensure_scalar_len

*Function*

Rejects a scalar put whose buffer already exceeds `limit`.

# Errors

Returns `StorageError::PayloadTooLarge` when `len` is above `limit`.

```rust
fn ensure_scalar_len(len: usize, limit: u64) -> crate::error::Result<()>
```



## bookclerk_storage::bounded::normalize_range

*Function*

Normalizes an internal range.

`None` reads the whole object. `ByteRange::length` `None` reads through
EOF (the Cap'n Proto wire uses integer `0` for that case and decodes it to
`None` before it reaches a backend). `Some(0)` is rejected so local and S3
cannot disagree about an empty window versus "to end".

`object_size`, when known, is the whole object. Offsets past that size, a
window that would extend past it, and `offset + length` overflow are errors.

# Errors

Returns `StorageError::InvalidKey` for an ambiguous, overflowing, or
unsatisfiable range.

```rust
fn normalize_range(range: Option<crate::traits::ByteRange>, object_size: Option<u64>) -> crate::error::Result<Option<ReadSpan>>
```



## bookclerk_storage::bounded::parse_sha256_hex

*Function*

Parses a hex SHA-256 (64 hex chars) into raw bytes.

# Errors

Returns `StorageError::Integrity` when the shape is wrong.

```rust
fn parse_sha256_hex(value: &str) -> crate::error::Result<[u8; 32]>
```



## bookclerk_storage::bounded::read_scalar_body

*Function*

Reads at most `limit + 1` bytes.

A body longer than `limit` returns `StorageError::PayloadTooLarge` and
stops. The allocation is capped at `limit + 1` regardless of any advertised
content length.

# Errors

Returns `StorageError::PayloadTooLarge` or `StorageError::Io`.

```rust
fn read_scalar_body(reader: std::pin::Pin<Box<dyn AsyncRead>>, limit: u64) -> crate::error::Result<bytes::Bytes>
```



## bookclerk_storage::bounded::reject_scalar_hint

*Function*

Early-reject when a HEAD/stat hint already exceeds `limit`.

This does not allocate `hint` bytes. The body must still be counted.

# Errors

Returns `StorageError::PayloadTooLarge` when `hint` is above `limit`.

```rust
fn reject_scalar_hint(hint: u64, limit: u64) -> crate::error::Result<()>
```



## bookclerk_storage::bounded::sha256_field_from_raw

*Function*

Validates a raw SHA-256 field (ABI `Data`, 32 bytes) into hex.

Empty means "not provided". Any other length is an integrity error.

# Errors

Returns `StorageError::Integrity` when the digest is not 32 bytes.

```rust
fn sha256_field_from_raw(bytes: Option<&[u8]>) -> crate::error::Result<Option<String>>
```



