**bookclerk_mp4 > parser**

# Module: parser

## Contents

**Structs**

- [`AudioTrack`](#audiotrack) - Parsed audio track summary (timescale, samples, codec).
- [`Mp4File`](#mp4file) - Parsed progressive (non-fragmented) MP4 relevant to remux.
- [`Mp4aConfig`](#mp4aconfig) - Clear AAC (`mp4a`) decoder config extracted from a progressive MP4/M4A/M4B.

**Enums**

- [`SampleEntryKind`](#sampleentrykind) - Codec or encryption FourCC found in the track's sample description.

**Functions**

- [`extract_mp4a_config`](#extract_mp4a_config) - Read sample-rate / channel count / ASC from a clear `mp4a` sample entry.
- [`parse_mp4`](#parse_mp4) - Parses top-level boxes and the primary audio track from an MP4/M4A/M4B file.
- [`track_duration_ms`](#track_duration_ms) - Absolute duration of the audio track in milliseconds.

---

## bookclerk_mp4::parser::AudioTrack

*Struct*

Parsed audio track summary (timescale, samples, codec).

**Fields:**
- `trak: crate::boxutil::BoxHeader` - ISO-BMFF `trak` (track) FourCC.
- `timescale: u32` - Media timescale (ticks per second) for this track.
- `duration: u64` - Track duration in media timescale ticks.
- `sample_entry_kind: SampleEntryKind` - Codec or encryption FourCC found in the track's sample description.
- `sample_entry_type_offset: u64` - Absolute file offset of the 4-byte sample-entry type inside stsd.
- `samples: Vec<crate::samples::SampleInfo>` - Decoded sample table (offsets, sizes, durations) for this track.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::parser::Mp4File

*Struct*

Parsed progressive (non-fragmented) MP4 relevant to remux.

**Fields:**
- `path: std::path::PathBuf` - Absolute path to the MP4 file that was parsed.
- `file_size: u64` - Total file size in bytes.
- `ftyp: crate::boxutil::BoxHeader` - ISO-BMFF `ftyp` (file type) FourCC.
- `moov: crate::boxutil::BoxHeader` - ISO-BMFF `moov` (movie metadata) FourCC.
- `mdat: crate::boxutil::BoxHeader` - ISO-BMFF `mdat` (media data) FourCC.
- `major_brand: crate::boxutil::FourCC` - Primary brand FourCC from `ftyp` (e.g. `M4A `, `isom`).
- `compatible_brands: Vec<crate::boxutil::FourCC>` - Compatible brand FourCCs advertised in `ftyp`.
- `mvhd_timescale: u32` - Movie-header timescale (ticks per second).
- `mvhd_duration: u64` - Movie-header duration in `Self::mvhd_timescale` ticks.
- `audio: AudioTrack` - Primary audio track summary when present.
- `moov_bytes: Vec<u8>` - Raw moov bytes (including header) — used when rewriting with patched stsd.
- `ftyp_bytes: Vec<u8>` - Raw `ftyp` box bytes (including header) for remux / patch.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::parser::Mp4aConfig

*Struct*

Clear AAC (`mp4a`) decoder config extracted from a progressive MP4/M4A/M4B.

**Fields:**
- `sample_rate: u32` - Audio sample rate in Hz from the sample entry.
- `channels: u16` - Channel count from the sample entry.
- `asc: Vec<u8>` - AudioSpecificConfig from `esds` DecoderSpecificInfo.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Mp4aConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::parser::SampleEntryKind

*Enum*

Codec or encryption FourCC found in the track's sample description.

**Variants:**
- `Aavd` - Audible Adrm encrypted AAC (`aavd`).
- `Mp4a` - Clear AAC (`mp4a`).
- `Enca` - Common Encryption audio (`enca`).
- `Other(crate::boxutil::FourCC)` - Opaque error wrapped from `anyhow`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &SampleEntryKind) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SampleEntryKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::parser::extract_mp4a_config

*Function*

Read sample-rate / channel count / ASC from a clear `mp4a` sample entry.

# Arguments

* `mp4` - `mp4` input for this call.

# Returns

On success, the inner `Mp4aConfig` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn extract_mp4a_config(mp4: &Mp4File) -> crate::error::Result<Mp4aConfig>
```



## bookclerk_mp4::parser::parse_mp4

*Function*

Parses top-level boxes and the primary audio track from an MP4/M4A/M4B file.

# Arguments

* `path` - Filesystem path involved in this operation.

# Returns

On success, the inner `Mp4File` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn parse_mp4(path: &std::path::Path) -> crate::error::Result<Mp4File>
```



## bookclerk_mp4::parser::track_duration_ms

*Function*

Absolute duration of the audio track in milliseconds.

# Arguments

* `track` - `track` input for this call.

# Returns

`u64` result.

```rust
fn track_duration_ms(track: &AudioTrack) -> u64
```



