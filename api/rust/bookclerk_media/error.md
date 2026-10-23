**bookclerk_media > error**

# Module: error

## Contents

**Enums**

- [`MediaError`](#mediaerror) - Errors from in-process codecs, MP4 plumbing, and confined media workers.

**Type Aliases**

- [`Result`](#result) - Result alias for media packaging and worker operations in this crate.

---

## bookclerk_media::error::MediaError

*Enum*

Errors from in-process codecs, MP4 plumbing, and confined media workers.

**Variants:**
- `InputMissing(std::path::PathBuf)` - A declared input path does not exist or is unreachable.
- `OutputMissing(std::path::PathBuf)` - The job reported success but the expected output path was not written.
- `Mp4(String)` - ISO-BMFF / MP4 container parse or remux failure.
- `Native(String)` - Native encode / remux / metadata failure (non-container detail).
- `Worker{ job: &'static str, detail: String }` - A media worker process could not be started, died, or returned a reply
- `NotIsolated{ job: &'static str, detail: String }` - Isolation is required but unavailable, so the job was refused rather
- `Io(std::io::Error)` - Local filesystem I/O failure.
- `Other(anyhow::Error)` - Catch-all for wrapped media failures.

**Trait Implementations:**

- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **From**
  - `fn from(err: bookclerk_mp4::Mp4Error) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_media::error::Result

*Type Alias*: `std::result::Result<T, MediaError>`

Result alias for media packaging and worker operations in this crate.



