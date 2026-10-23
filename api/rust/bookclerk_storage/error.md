**bookclerk_storage > error**

# Module: error

## Contents

**Enums**

- [`StorageError`](#storageerror) - Errors from local / S3 / fan-out storage backends.

**Type Aliases**

- [`Result`](#result) - Result alias for storage backend operations in this crate.

---

## bookclerk_storage::error::Result

*Type Alias*: `std::result::Result<T, StorageError>`

Result alias for storage backend operations in this crate.



## bookclerk_storage::error::StorageError

*Enum*

Errors from local / S3 / fan-out storage backends.

**Variants:**
- `NotFound(String)` - Object key was not found on get/probe/copy source.
- `InvalidKey(String)` - Key failed validation (path escape, empty bucket, bad destination config).
- `Io(std::io::Error)` - Local filesystem I/O failure.
- `S3(String)` - S3 SDK or HTTP failure (operator-facing detail).
- `PayloadTooLarge(String)` - Scalar get/put exceeded the ABI small-object cap.
- `Integrity(String)` - Declared length or SHA-256 did not match the bytes consumed.
- `InvalidCursor(String)` - List cursor is missing, stale, or not from this backend.
- `Other(anyhow::Error)` - Catch-all for wrapped backend failures.

**Trait Implementations:**

- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



