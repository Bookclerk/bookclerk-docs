**bookclerk_media > package_m4b**

# Module: package_m4b

## Contents

**Structs**

- [`PackageM4bRequest`](#packagem4brequest) - Request to package ordered MP3 parts into one M4B.

**Functions**

- [`package_m4b_from_mp3`](#package_m4b_from_mp3) - Decode/remux ordered parts → progressive M4B.
- [`package_m4b_from_pcm`](#package_m4b_from_pcm) - Encode interleaved PCM to AAC-LC and mux an M4B (test / internal helper).

---

## bookclerk_media::package_m4b::PackageM4bRequest

*Struct*

Request to package ordered MP3 parts into one M4B.

**Fields:**
- `parts: Vec<std::path::PathBuf>` - Ordered MP3 chapter / part files.
- `output: std::path::PathBuf` - Destination `.m4b` path.
- `chapter_titles: Vec<String>` - Optional chapter titles (length may match `parts`).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PackageM4bRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_media::package_m4b::package_m4b_from_mp3

*Function*

Decode/remux ordered parts → progressive M4B.

Clear AAC MP4/M4A/M4B parts are remuxed without re-encoding. MP3 parts are
decoded and encoded to AAC-LC via a streaming path.

Returns the output path plus chapter list `(title, start_ms)`.
Runs in a confined media worker; see the [crate](bookclerk_media.md) documentation.

# Errors

Returns `MediaError::InputMissing` when a part is missing, and propagates
packaging and worker failures otherwise.

```rust
fn package_m4b_from_mp3(req: PackageM4bRequest) -> crate::error::Result<(crate::MediaOutcome, Vec<(String, u64)>)>
```



## bookclerk_media::package_m4b::package_m4b_from_pcm

*Function*

Encode interleaved PCM to AAC-LC and mux an M4B (test / internal helper).

Feeds PCM in bounded chunks so callers with large buffers still do not
duplicate the full stream as AAC access units in memory.

# Errors

Returns an error when the operation fails.

```rust
fn package_m4b_from_pcm(pcm: &[i16], sample_rate: u32, channels: u16, output: &std::path::Path, chapter_boundaries_ms: &[(String, u64)]) -> crate::error::Result<(crate::MediaOutcome, Vec<(String, u64)>)>
```



