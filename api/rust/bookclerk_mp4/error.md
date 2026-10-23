**bookclerk_mp4 > error**

# Module: error

## Contents

**Enums**

- [`Mp4Error`](#mp4error) - Errors from MP4 parse, remux, and in-place patch operations.

**Type Aliases**

- [`Result`](#result) - Result alias for this crate's error type.

---

## bookclerk_mp4::error::Mp4Error

*Enum*

Errors from MP4 parse, remux, and in-place patch operations.

**Variants:**
- `Container(String)` - The file does not match the shape this crate can read or write.
- `Transform(String)` - A [`crate::SampleTransform`](remux.md#bookclerk_mp4remuxsampletransform) refused a payload. The crate never produces
- `NoRoom{ needed: usize, available: usize }` - A rebuilt `moov` outgrew the space reserved for it, so it cannot be
- `Io(std::io::Error)` - Underlying filesystem or stream I/O failure.

**Methods:**

- `fn container<impl Into<String>>(detail: impl Trait) -> Self` - Shorthand for `Mp4Error::Container`.
- `fn transform<impl Into<String>>(detail: impl Trait) -> Self` - Shorthand for `Mp4Error::Transform`.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`



## bookclerk_mp4::error::Result

*Type Alias*: `std::result::Result<T, Mp4Error>`

Result alias for this crate's error type.



