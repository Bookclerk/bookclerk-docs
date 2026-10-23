**bookclerk_acquire > convert**

# Module: convert

## Contents

**Structs**

- [`ConvertRequest`](#convertrequest) - Options for [`convert_book`](convert.md#bookclerk_acquireconvertconvert_book).
- [`ConvertSummary`](#convertsummary) - Summary of a batch convert run.

**Functions**

- [`convert_book`](#convert_book) - Convert one acquired m4b/m4a to mp3 and update the library storage key.

---

## bookclerk_acquire::convert::ConvertRequest

*Struct*

Options for [`convert_book`](convert.md#bookclerk_acquireconvertconvert_book).

**Fields:**
- `cache_dir: std::path::PathBuf` - Scratch directory for temporary acquire/convert files.
- `force: bool` - When true, re-download or re-convert even if output exists.
- `lame: bookclerk_config::LameConfig` - LAME MP3 encoder settings from config.
- `max_sample_rate: Option<u32>` - Optional ceiling on output sample rate in Hz.
- `job_id: Option<String>` - Durable job that owns the scratch reservation, when this convert is queued.
- `temp_quota_bytes: Option<u64>` - Byte budget for this conversion's input and output files together.
- `cancel: Option<std::sync::Arc<std::sync::atomic::AtomicBool>>` - Set when the caller wants the copy loop to stop.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ConvertRequest`



## bookclerk_acquire::convert::ConvertSummary

*Struct*

Summary of a batch convert run.

**Fields:**
- `converted: u32` - Titles successfully converted in this run.
- `skipped: u32` - Titles skipped (already done or ineligible).
- `failed: u32` - Titles that failed conversion or matching.

**Trait Implementations:**

- **Default**
  - `fn default() -> ConvertSummary`
- **Clone**
  - `fn clone(self: &Self) -> ConvertSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::convert::convert_book

*Function*

Convert one acquired m4b/m4a to mp3 and update the library storage key.

# Arguments

* `library` - Library store used to update acquire status / storage key.
* `storage` - Object storage backend holding the source and destination objects.
* `book` - Acquired book row whose `storage_key` points at m4b/m4a.
* `req` - Cache directory, force flag, and LAME settings.

# Returns

Object-storage key of the written MP3.

# Errors

Returns [`AcquireError`](error.md#bookclerk_acquireerroracquireerror) when the source is missing/ineligible, encode fails,
or library/storage updates fail.

```rust
fn convert_book(library: &bookclerk_library::LibraryStore, storage: &dyn StorageBackend, book: &bookclerk_library::BookRecord, req: &ConvertRequest) -> crate::error::Result<String>
```



