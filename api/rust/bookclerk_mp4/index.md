# bookclerk_mp4

ISO-BMFF (MP4) container plumbing: read a progressive file, pick samples by
media time, and write a faststart `ftyp` + `moov` + `mdat` M4B in one pass.

## No cryptography here

[`remux_progressive`](#remux_progressive) streams payloads through a [`SampleTransform`](#sampletransform), and the
only implementation this crate ships is [`CopySamples`](#copysamples). A store that has to
turn its own ciphertext into plaintext keeps both the key and the cipher on
its side of that trait, inside its own plugin process.

That split is why two very similar remuxers used to exist: the host needed
one for clear media and the Audible plugin needed one that could decrypt
mid-copy, and neither could take on the other's dependencies. The transform
hook removes the copy without moving the boundary — see `docs/server/media.md` for
why the boundary is where it is.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SampleTransform`](#sampletransform) | trait | Rewrites sample payloads as they stream from input to output. |
| [`AudioTrack`](#audiotrack) | struct | Parsed audio track summary (timescale, samples, codec). |
| [`Mp4File`](#mp4file) | struct | Parsed progressive (non-fragmented) MP4 relevant to remux. |
| [`Mp4aConfig`](#mp4aconfig) | struct | Clear AAC (`mp4a`) decoder config extracted from a progressive MP4/M4A/M4B. |
| [`MoovLocation`](#moovlocation) | struct | Where a file's `moov` sits. |
| [`SampleReader`](#samplereader) | struct | Reads sample payloads by absolute file offset, buffered. |
| [`CopySamples`](#copysamples) | struct | Copies every sample through untouched. |
| [`ProgressiveWriteInput`](#progressivewriteinput) | struct | Inputs for writing a progressive faststart M4B (ftyp + moov + mdat) in one pass. |
| [`RemuxOptions`](#remuxoptions) | struct | Options controlling sample-copy remux (trim window, etc.). |
| [`TrimRange`](#trimrange) | struct | Optional media-time trim window in milliseconds (absolute, pre-rebase). |
| [`ChunkMapEntry`](#chunkmapentry) | struct | One run from the sample-to-chunk (`stsc`) table. |
| [`SampleInfo`](#sampleinfo) | struct | One audio sample, located and timed. |
| [`Mp4Error`](#mp4error) | enum | Errors from MP4 parse, remux, and in-place patch operations. |
| [`SampleEntryKind`](#sampleentrykind) | enum | Codec or encryption FourCC found in the track's sample description. |
| [`extract_mp4a_config`](#extract_mp4a_config) | function | Read sample-rate / channel count / ASC from a clear `mp4a` sample entry. |
| [`parse_mp4`](#parse_mp4) | function | Parses top-level boxes and the primary audio track from an MP4/M4A/M4B file. |
| [`track_duration_ms`](#track_duration_ms) | function | Absolute duration of the audio track in milliseconds. |
| [`pad_moov_to`](#pad_moov_to) | function | Grow `moov` to exactly `total` bytes by appending a `free` child. |
| [`read_moov`](#read_moov) | function | Read a file's `moov`, and note where it sits. |
| [`shift_chunk_offsets`](#shift_chunk_offsets) | function | Move every chunk offset in `moov` past `above` by `delta` bytes. |
| [`strip_trailing_free`](#strip_trailing_free) | function | Drop a `moov`'s trailing `free` child, if it has one. |
| [`top_level_boxes`](#top_level_boxes) | function | Every box at the top level, in file order. |
| [`write_moov_in_place`](#write_moov_in_place) | function | Write `moov` back where the old one was, padding it to the same length. |
| [`remux_progressive`](#remux_progressive) | function | Copy (and optionally trim) a progressive MP4 into a faststart M4B, passing every payload through `transform`. |
| [`write_progressive_m4b`](#write_progressive_m4b) | function | Write a progressive faststart M4B by streaming one sample at a time. |
| [`build_samples`](#build_samples) | function | Builds a flat sample table from `stts` / `stsc` / `stsz` / `stco` (or `co64`). |
| [`select_samples_by_ms`](#select_samples_by_ms) | function | Indices of the samples whose media time overlaps `[start_ms, end_ms)`. |
| [`Result`](#result) | type alias | Result alias for this crate's error type. |
| [`RESERVED_MOOV_SLACK`](#reserved_moov_slack) | constant | Slack every `moov` this crate writes carries, as a trailing `free` child. |

## Modules

- [`boxutil`](boxutil.md): Low-level big-endian readers and MP4 box walking.
- [`edit`](edit.md): Editing a `moov` that is already in memory: find a box, replace a child, and keep every enclosing size field consistent.
- [`fixture`](fixture.md): Synthetic progressive MP4 files for tests.

## Traits

### SampleTransform

```rust
pub trait SampleTransform {
    fn retain(&mut self, kept: &[usize]) -> Result<()>;
    fn sample(&mut self, index: usize, payload: &mut [u8]) -> Result<()>;
}
```

Rewrites sample payloads as they stream from input to output.

The remuxer moves bytes and rebuilds tables; it never inspects a payload.
A caller that has to turn its own ciphertext into plaintext keeps the key and
the cipher on its side of this trait, which is how store plugins decrypt
their downloads without any of that living here. [`CopySamples`](#copysamples) is the only
implementation in this crate.

#### SampleTransform::retain

```rust
fn retain(&mut self, kept: &[usize]) -> Result<()>
```

The original sample indices kept by the trim, in output order.

Called once before any payload, so a transform holding per-sample state
(a table of initialization vectors, say) can narrow it to the same
selection and then index it by output position.

##### Errors

Returns an error when the operation fails.

#### SampleTransform::sample

```rust
fn sample(&mut self, index: usize, payload: &mut [u8]) -> Result<()>
```

Rewrite output sample `index` in place. The length must not change.

##### Errors

Returns an error when the operation fails.

Implemented by: `CopySamples`

## Structs

### AudioTrack

```rust
pub struct AudioTrack {
    pub trak: BoxHeader,
    pub timescale: u32,
    pub duration: u64,
    pub sample_entry_kind: SampleEntryKind,
    pub sample_entry_type_offset: u64,
    pub samples: Vec<SampleInfo>,
}
```

Parsed audio track summary (timescale, samples, codec).

**Fields**

- `trak`: ISO-BMFF `trak` (track) FourCC.
- `timescale`: Media timescale (ticks per second) for this track.
- `duration`: Track duration in media timescale ticks.
- `sample_entry_kind`: Codec or encryption FourCC found in the track's sample description.
- `sample_entry_type_offset`: Absolute file offset of the 4-byte sample-entry type inside stsd.
- `samples`: Decoded sample table (offsets, sizes, durations) for this track.

Implements: `Debug`

### Mp4File

```rust
pub struct Mp4File {
    pub path: PathBuf,
    pub file_size: u64,
    pub ftyp: BoxHeader,
    pub moov: BoxHeader,
    pub mdat: BoxHeader,
    pub major_brand: FourCC,
    pub compatible_brands: Vec<FourCC>,
    pub mvhd_timescale: u32,
    pub mvhd_duration: u64,
    pub audio: AudioTrack,
    pub moov_bytes: Vec<u8>,
    pub ftyp_bytes: Vec<u8>,
}
```

Parsed progressive (non-fragmented) MP4 relevant to remux.

**Fields**

- `path`: Absolute path to the MP4 file that was parsed.
- `file_size`: Total file size in bytes.
- `ftyp`: ISO-BMFF `ftyp` (file type) FourCC.
- `moov`: ISO-BMFF `moov` (movie metadata) FourCC.
- `mdat`: ISO-BMFF `mdat` (media data) FourCC.
- `major_brand`: Primary brand FourCC from `ftyp` (e.g. `M4A `, `isom`).
- `compatible_brands`: Compatible brand FourCCs advertised in `ftyp`.
- `mvhd_timescale`: Movie-header timescale (ticks per second).
- `mvhd_duration`: Movie-header duration in [`Self::mvhd_timescale`](#mp4file) ticks.
- `audio`: Primary audio track summary when present.
- `moov_bytes`: Raw moov bytes (including header) — used when rewriting with patched stsd.
- `ftyp_bytes`: Raw `ftyp` box bytes (including header) for remux / patch.

Implements: `Debug`

### Mp4aConfig

```rust
pub struct Mp4aConfig {
    pub sample_rate: u32,
    pub channels: u16,
    pub asc: Vec<u8>,
}
```

Clear AAC (`mp4a`) decoder config extracted from a progressive MP4/M4A/M4B.

**Fields**

- `sample_rate`: Audio sample rate in Hz from the sample entry.
- `channels`: Channel count from the sample entry.
- `asc`: AudioSpecificConfig from `esds` DecoderSpecificInfo.

Implements: `Clone`, `Debug`

### MoovLocation

```rust
pub struct MoovLocation {
    pub start: u64,
    pub len: u64,
    pub is_last: bool,
}
```

Where a file's `moov` sits.

**Fields**

- `start`: Byte offset of this box or region within the file.
- `len`: Total box length, `free` padding included.
- `is_last`: Whether `moov` is the last box in the file, in which case it can grow
  past its slack for the price of an append.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SampleReader

```rust
pub struct SampleReader {
    /* private fields */
}
```

Reads sample payloads by absolute file offset, buffered.

Samples sit in track order in a progressive `mdat`, so consecutive reads are
usually already in place, and otherwise a short hop ahead. Both cases are
served out of the buffer; only a jump past it costs a real seek. Reading in
some other order still works, just without the benefit.

#### SampleReader::open

```rust
pub fn open(path: &Path) -> Result<Self>
```

Opens `path` for buffered sample reads.

##### Arguments

* `path` - Filesystem path to a progressive MP4/M4A/M4B.

##### Returns

Reader positioned at byte 0.

##### Errors

Returns [`crate::Mp4Error::Io`](#mp4error) when the file cannot be opened.

#### SampleReader::new

```rust
pub fn new(file: File) -> Self
```

Wraps an already-opened file with a buffered sample reader.

##### Arguments

* `file` - Seekable file handle positioned anywhere (reader seeks per sample).

##### Returns

Buffered reader ready for [`Self::read_sample`](#samplereaderread_sample).

#### SampleReader::read_sample

```rust
pub fn read_sample(&mut self, offset: u64, size: usize, buf: &mut Vec<u8>) -> Result<()>
```

Fill `buf` with exactly `size` bytes from `offset`, resizing it to match.

##### Arguments

* `offset` - Absolute byte offset in the file.
* `size` - Number of bytes to read.
* `buf` - Destination buffer resized to `size`.

##### Returns

The successful result value for this operation.

##### Errors

Returns [`Mp4Error::Container`](#mp4error) when the sample would end past the end of
the file (a forged or truncated sample table), and an I/O error when the
read itself fails.

Implements: `Debug`

### CopySamples

```rust
pub struct CopySamples;
```

Copies every sample through untouched.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `SampleTransform`

### ProgressiveWriteInput

```rust
pub struct ProgressiveWriteInput<'a> {
    pub moov_bytes: &'a [u8],
    pub moov_file_start: u64,
    pub sample_entry_type_offset: u64,
    pub audio_timescale: u32,
    pub mvhd_timescale: u32,
    pub sample_sizes: &'a [u32],
    pub durations: &'a [u32],
}
```

Inputs for writing a progressive faststart M4B (ftyp + moov + mdat) in one pass.

**Fields**

- `moov_bytes`: Raw `moov` bytes used while rewriting sample tables.
- `moov_file_start`: File offset where the `moov` box begins.
- `sample_entry_type_offset`: File offset of the sample-entry FourCC inside `stsd`.
- `audio_timescale`: Audio track media timescale (ticks per second).
- `mvhd_timescale`: Movie-header timescale (ticks per second).
- `sample_sizes`: Per-sample byte sizes copied into the output.
- `durations`: Per-sample durations in media timescale ticks.

### RemuxOptions

```rust
pub struct RemuxOptions {
    pub trim: Option<TrimRange>,
}
```

Options controlling sample-copy remux (trim window, etc.).

**Fields**

- `trim`: Optional start/end trim window applied during remux.

Implements: `Clone`, `Debug`, `Default`

### TrimRange

```rust
pub struct TrimRange {
    pub start_ms: u64,
    pub end_ms: Option<u64>,
}
```

Optional media-time trim window in milliseconds (absolute, pre-rebase).

**Fields**

- `start_ms`: Trim start offset in milliseconds from the beginning of the audio.
- `end_ms`: Exclusive end; `None` means "through end of media".

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### ChunkMapEntry

```rust
pub struct ChunkMapEntry {
    pub first_chunk: u32,
    pub samples_per_chunk: u32,
    pub sample_description_index: u32,
}
```

One run from the sample-to-chunk (`stsc`) table.

**Fields**

- `first_chunk`: 1-based first chunk index for this `stsc` entry.
- `samples_per_chunk`: Number of samples in each chunk of this run.
- `sample_description_index`: 1-based index into the sample description table.

Implements: `Clone`, `Debug`

### SampleInfo

```rust
pub struct SampleInfo {
    pub offset: u64,
    pub size: u32,
    pub start_cts: u64,
    pub duration: u32,
    pub chunk_index: u32,
}
```

One audio sample, located and timed.

**Fields**

- `offset`: Absolute file offset of the sample payload.
- `size`: Total box size in bytes including the header.
- `start_cts`: Composition start time in media timescale ticks.
- `duration`: Sample duration in media timescale ticks.
- `chunk_index`: 1-based chunk index that contains this sample.

Implements: `Clone`, `Debug`

## Enums

### Mp4Error

```rust
pub enum Mp4Error {
    Container(String),
    Transform(String),
    NoRoom { needed: usize, available: usize },
    Io(Error),
}
```

Errors from MP4 parse, remux, and in-place patch operations.

**Variants**

- `Container`: The file does not match the shape this crate can read or write.
- `Transform`: A [`crate::SampleTransform`](#sampletransform) refused a payload. The crate never produces
  this itself; it carries the caller's reason out through the remuxer.
- `NoRoom`: A rebuilt `moov` outgrew the space reserved for it, so it cannot be
  swapped in without moving the media. Callers fall back to a rewrite.
  - `needed`: Bytes required for the rebuilt `moov`.
  - `available`: Bytes currently reserved for in-place swap.
- `Io`: Underlying filesystem or stream I/O failure.

#### Mp4Error::container

```rust
pub fn container(detail: impl Into<String>) -> Self
```

Shorthand for [`Mp4Error::Container`](#mp4error).

##### Arguments

* `detail` - Human-readable error detail.

##### Returns

Updated `Self` for chaining.

#### Mp4Error::transform

```rust
pub fn transform(detail: impl Into<String>) -> Self
```

Shorthand for [`Mp4Error::Transform`](#mp4error).

##### Arguments

* `detail` - Human-readable error detail.

##### Returns

Updated `Self` for chaining.

Implements: `Debug`, `Display`, `Error`, `From<Error>`

### SampleEntryKind

```rust
pub enum SampleEntryKind {
    Aavd,
    Mp4a,
    Enca,
    Other(FourCC),
}
```

Codec or encryption FourCC found in the track's sample description.

**Variants**

- `Aavd`: Audible Adrm encrypted AAC (`aavd`).
- `Mp4a`: Clear AAC (`mp4a`).
- `Enca`: Common Encryption audio (`enca`).
- `Other`: Opaque error wrapped from `anyhow`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### extract_mp4a_config

```rust
pub fn extract_mp4a_config(mp4: &Mp4File) -> Result<Mp4aConfig>
```

Read sample-rate / channel count / ASC from a clear `mp4a` sample entry.

#### Arguments

* `mp4` - `mp4` input for this call.

#### Returns

On success, the inner `Mp4aConfig` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

### parse_mp4

```rust
pub fn parse_mp4(path: &Path) -> Result<Mp4File>
```

Parses top-level boxes and the primary audio track from an MP4/M4A/M4B file.

#### Arguments

* `path` - Filesystem path involved in this operation.

#### Returns

On success, the inner `Mp4File` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### track_duration_ms

```rust
pub fn track_duration_ms(track: &AudioTrack) -> u64
```

Absolute duration of the audio track in milliseconds.

#### Arguments

* `track` - `track` input for this call.

#### Returns

`u64` result.

### pad_moov_to

```rust
pub fn pad_moov_to(moov: &mut Vec<u8>, total: usize) -> Result<()>
```

Grow `moov` to exactly `total` bytes by appending a `free` child.

#### Errors

[`Mp4Error::NoRoom`](#mp4error) when `moov` is already longer than `total`, or when the
gap is too small to express — a `free` box cannot be shorter than its own
header, so one to seven spare bytes are as unusable as none.

#### Panics

Panics when an internal invariant does not hold.

### read_moov

```rust
pub fn read_moov(path: &Path) -> Result<(MoovLocation, Vec<u8>)>
```

Read a file's `moov`, and note where it sits.

Only the box headers and `moov` itself are read, so the cost is the header's
and not the file's.

#### Arguments

* `path` - Filesystem path involved in this operation.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### shift_chunk_offsets

```rust
pub fn shift_chunk_offsets(moov: &mut [u8], above: u64, delta: i64) -> Result<()>
```

Move every chunk offset in `moov` past `above` by `delta` bytes.

Chunk offsets are absolute file positions, so an edit that shifts the media
has to correct them — and an edit that only *appeared* to shift it, because
the work happened in a scratch file laid out differently, has to correct them
back. Only what sits after the edit moves, which is why `above` is usually
where `moov` starts: media in front of the header stays put.

#### Arguments

* `moov` - `moov` input for this call.
* `above` - Numeric `above` value for this call.
* `delta` - Numeric `delta` value for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

### strip_trailing_free

```rust
pub fn strip_trailing_free(moov: &mut Vec<u8>) -> Result<bool>
```

Drop a `moov`'s trailing `free` child, if it has one.

Anything appended to `moov` goes after its last child, so the slack has to
come off before an edit and go back on after it. Returns whether one was
there to remove.

#### Arguments

* `moov` - `moov` input for this call.

#### Returns

On success, the inner `bool` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### top_level_boxes

```rust
pub fn top_level_boxes(file: &mut File) -> Result<Vec<BoxHeader>>
```

Every box at the top level, in file order.

#### Arguments

* `file` - Open file handle.

#### Returns

On success, the inner `Vec<BoxHeader>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### write_moov_in_place

```rust
pub fn write_moov_in_place(path: &Path, at: MoovLocation, moov: &[u8]) -> Result<()>
```

Write `moov` back where the old one was, padding it to the same length.

The media does not move, so every chunk offset in the file stays true.

#### Errors

[`Mp4Error::NoRoom`](#mp4error) when the rebuilt `moov` does not fit the space the old
one occupied and cannot simply be appended to.

### remux_progressive

```rust
pub fn remux_progressive(
    input: &Path,
    output: &Path,
    opts: &RemuxOptions,
    transform: &mut dyn SampleTransform,
) -> Result<()>
```

Copy (and optionally trim) a progressive MP4 into a faststart M4B, passing
every payload through `transform`.

Sample *payloads* are streamed one buffer at a time. Only the retained sizes,
durations, and read offsets are held in memory; the parsed sample table is
never cloned.

#### Arguments

* `input` - Candidate score input from the public catalog.
* `output` - Filesystem path (`output`).
* `opts` - Options struct for this operation.
* `transform` - `transform` input for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### write_progressive_m4b

```rust
pub fn write_progressive_m4b<F>(
    output: &Path,
    input: ProgressiveWriteInput<'_>,
    fill_sample: F,
) -> Result<()>
where
    F: FnMut(usize, &mut Vec<u8>) -> Result<()>,
```

Write a progressive faststart M4B by streaming one sample at a time.

Layout is written as `ftyp` + `moov` + `mdat` in a single pass (no full-file
rewrite), and `ftyp` always declares M4B brands. `fill_sample(i, buf)` must
populate `buf` with the payload for sample `i` (length must match
`sample_sizes[i]`). Only one sample buffer is live at a time.

Callers that assemble their own sample plan — from fragments, say, rather
than from one progressive `mdat` — use this directly.

#### Arguments

* `output` - Filesystem path (`output`).
* `input` - Candidate score input from the public catalog.
* `fill_sample` - `fill_sample` input for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### build_samples

```rust
pub fn build_samples(
    stts: &[(u32, u32)],
    stsc: &[ChunkMapEntry],
    sample_sizes: &[u32],
    chunk_offsets: &[u64],
) -> Result<Vec<SampleInfo>>
```

Builds a flat sample table from `stts` / `stsc` / `stsz` / `stco` (or `co64`).

#### Arguments

* `stts` - Numeric `stts` value for this call.
* `stsc` - `stsc` input for this call.
* `sample_sizes` - Numeric `sample_sizes` value for this call.
* `chunk_offsets` - Numeric `chunk_offsets` value for this call.

#### Returns

On success, the inner `Vec<SampleInfo>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

### select_samples_by_ms

```rust
pub fn select_samples_by_ms(
    samples: &[SampleInfo],
    timescale: u32,
    start_ms: u64,
    end_ms: Option<u64>,
) -> Vec<usize>
```

Indices of the samples whose media time overlaps `[start_ms, end_ms)`.

Indices rather than clones: a long audiobook has millions of samples, and the
caller needs the original positions anyway to line up any per-sample state it
keeps of its own. Output composition times are rebased by the writer.

#### Arguments

* `samples` - `samples` input for this call.
* `timescale` - Numeric `timescale` value for this call.
* `start_ms` - Numeric `start_ms` value for this call.
* `end_ms` - Numeric `end_ms` value for this call.

#### Returns

Collected results (may be empty).

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, Mp4Error>;
```

Result alias for this crate's error type.

## Constants

### RESERVED_MOOV_SLACK

```rust
pub const RESERVED_MOOV_SLACK: usize = _;
```

Slack every `moov` this crate writes carries, as a trailing `free` child.

A megabyte covers a full tag set, a cover, and a chapter track for a book
with several hundred chapters — the things that get added after the media is
already on disk. Against the file it protects, which is hundreds of megabytes
of audio, it costs about a thousandth; against the alternative, which is
rewriting all of that to make room, it costs nothing at all.
