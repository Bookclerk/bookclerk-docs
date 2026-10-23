**bookclerk_config > path_limits**

# Module: path_limits

## Contents

**Structs**

- [`PathLimits`](#pathlimits) - Limits applied when hardening acquired storage keys.

**Enums**

- [`PathLengthMeasure`](#pathlengthmeasure) - How to measure path component length when truncating.

**Functions**

- [`enforce_storage_key_limits`](#enforce_storage_key_limits) - Enforce per-segment and full-key limits on a relative storage key.
- [`path_len`](#path_len) - Measure `s` according to `measure`.
- [`truncate_filename_stem`](#truncate_filename_stem) - Truncate a filename stem so `stem + "." + ext` fits in `max_filename_length`.
- [`truncate_path_component`](#truncate_path_component) - Truncate `s` so its measured length is ≤ `limit`. Never splits a UTF-8 char.

**Constants**

- [`DEFAULT_MAX_FILENAME_LENGTH`](#default_max_filename_length) - Classic Libation `LongPath.MaxFilenameLength` — per path component.
- [`S3_MAX_OBJECT_KEY_BYTES`](#s3_max_object_key_bytes) - AWS S3 maximum object key length (UTF-8 bytes), including any prefix.

---

## bookclerk_config::path_limits::DEFAULT_MAX_FILENAME_LENGTH

*Constant*: `usize`

Classic Libation `LongPath.MaxFilenameLength` — per path component.



## bookclerk_config::path_limits::PathLengthMeasure

*Enum*

How to measure path component length when truncating.

**Variants:**
- `Chars` - Unicode scalar count (classic Libation on Windows).
- `Utf8Bytes` - UTF-8 byte count (classic Libation on Unix; always used for S3 keys).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PathLengthMeasure) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PathLengthMeasure`
- **Default**
  - `fn default() -> PathLengthMeasure`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::path_limits::PathLimits

*Struct*

Limits applied when hardening acquired storage keys.

**Fields:**
- `max_filename_length: usize` - Max length per folder / file-stem segment. `0` disables per-segment caps.
- `max_storage_key_bytes: usize` - Max UTF-8 bytes for the full relative storage key (`folder/…/file.ext`).
- `measure: PathLengthMeasure` - Whether segment truncation counts Unicode scalars or UTF-8 bytes.

**Methods:**

- `fn local(measure: PathLengthMeasure) -> Self` - Local-filesystem defaults: 255-length segments, no full-key cap.
- `fn s3(prefix: &str) -> Self` - S3 defaults: 255-byte segments plus a full-key budget of
- `fn resolve(max_filename_length: u32, storage_is_s3: bool, s3_prefix: &str, path_sanitization_is_windows: bool) -> Self` - Resolve from config: Windows sanitization → char measure; S3 → key budget.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PathLimits) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PathLimits`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::path_limits::S3_MAX_OBJECT_KEY_BYTES

*Constant*: `usize`

AWS S3 maximum object key length (UTF-8 bytes), including any prefix.



## bookclerk_config::path_limits::enforce_storage_key_limits

*Function*

Enforce per-segment and full-key limits on a relative storage key.

`key` uses `/` separators. The final component is treated as a filename
(extension preserved). Returns the possibly truncated key.

```rust
fn enforce_storage_key_limits(key: &str, limits: PathLimits) -> String
```



## bookclerk_config::path_limits::path_len

*Function*

Measure `s` according to `measure`.

```rust
fn path_len(s: &str, measure: PathLengthMeasure) -> usize
```



## bookclerk_config::path_limits::truncate_filename_stem

*Function*

Truncate a filename stem so `stem + "." + ext` fits in `max_filename_length`.

When the stem ends with ` [id]` (ASIN/ISBN bracket form used by naming
profiles), the id suffix is preserved and the title prefix is shortened.

```rust
fn truncate_filename_stem(stem: &str, ext: &str, max_filename_length: usize, measure: PathLengthMeasure) -> String
```



## bookclerk_config::path_limits::truncate_path_component

*Function*

Truncate `s` so its measured length is ≤ `limit`. Never splits a UTF-8 char.

```rust
fn truncate_path_component(s: &str, limit: usize, measure: PathLengthMeasure) -> String
```



