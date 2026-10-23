**bookclerk_storage > transfer**

# Module: transfer

## Contents

**Structs**

- [`TransferOptions`](#transferoptions) - Knobs for [`transfer_object`](transfer.md#bookclerk_storagetransfertransfer_object).

**Enums**

- [`TransferOutcome`](#transferoutcome) - How [`transfer_object`](transfer.md#bookclerk_storagetransfertransfer_object) moved the object.

**Functions**

- [`transfer_object`](#transfer_object) - Copies `source_key` onto `dest_key`.

---

## bookclerk_storage::transfer::TransferOptions

*Struct*

Knobs for [`transfer_object`](transfer.md#bookclerk_storagetransfertransfer_object).

**Fields:**
- `max_attempts: u32` - Total attempts including the first. Zero is treated as one.
- `backoff: std::time::Duration` - Base backoff between attempts. Attempt `n` waits `backoff * n`.
- `attempt_timeout: Option<std::time::Duration>` - Optional wall-clock budget for a single attempt.
- `stage_journal_dir: Option<std::path::PathBuf>` - Directory that records attempt-owned stages so a restarted process can

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> TransferOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_storage::transfer::TransferOutcome

*Enum*

How [`transfer_object`](transfer.md#bookclerk_storagetransfertransfer_object) moved the object.

**Variants:**
- `ServerCopy` - Destination `copy` inside one storage instance.
- `Streamed(crate::traits::PutStreamResult)` - Streamed body. `result.sha256_hex` is the digest of bytes written.
- `AlreadyPublished` - Destination probe already matched the source size and digest.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> TransferOutcome`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &TransferOutcome) -> bool`



## bookclerk_storage::transfer::transfer_object

*Function*

Copies `source_key` onto `dest_key`.

`meta` supplies content type and any caller-provided digest. A digest on
the source probe is required to match bytes actually written. When the
source probe has no digest, the destination records the digest computed
while streaming. An S3 ETag is an identity hint for "the object changed",
not a content checksum, and is never copied into `sha256_hex`.

# Errors

Returns `StorageError::Integrity` when the source changes between
attempts or a digest/length check fails, `StorageError::NotFound` when
the source is missing, and the destination error after retries are exhausted.

```rust
fn transfer_object(source: &dyn StorageBackend, source_key: &str, dest: &dyn StorageBackend, dest_key: &str, meta: crate::traits::ObjectMeta, options: &TransferOptions) -> crate::error::Result<TransferOutcome>
```



