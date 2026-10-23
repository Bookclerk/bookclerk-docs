**bookclerk_mp4 > remux**

# Module: remux

## Contents

**Structs**

- [`CopySamples`](#copysamples) - Copies every sample through untouched.
- [`ProgressiveWriteInput`](#progressivewriteinput) - Inputs for writing a progressive faststart M4B (ftyp + moov + mdat) in one pass.
- [`RemuxOptions`](#remuxoptions) - Options controlling sample-copy remux (trim window, etc.).
- [`TrimRange`](#trimrange) - Optional media-time trim window in milliseconds (absolute, pre-rebase).

**Functions**

- [`remux_progressive`](#remux_progressive) - Copy (and optionally trim) a progressive MP4 into a faststart M4B, passing
- [`write_progressive_m4b`](#write_progressive_m4b) - Write a progressive faststart M4B by streaming one sample at a time.

**Traits**

- [`SampleTransform`](#sampletransform) - Rewrites sample payloads as they stream from input to output.

---

## bookclerk_mp4::remux::CopySamples

*Struct*

Copies every sample through untouched.

**Unit Struct**

**Traits:** Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> CopySamples`
- **SampleTransform**
  - `fn sample(self: & mut Self, _index: usize, _payload: & mut [u8]) -> Result<()>`
- **Clone**
  - `fn clone(self: &Self) -> CopySamples`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::remux::ProgressiveWriteInput

*Struct*

Inputs for writing a progressive faststart M4B (ftyp + moov + mdat) in one pass.

**Generic Parameters:**
- 'a

**Fields:**
- `moov_bytes: &'a [u8]` - Raw `moov` bytes used while rewriting sample tables.
- `moov_file_start: u64` - File offset where the `moov` box begins.
- `sample_entry_type_offset: u64` - File offset of the sample-entry FourCC inside `stsd`.
- `audio_timescale: u32` - Audio track media timescale (ticks per second).
- `mvhd_timescale: u32` - Movie-header timescale (ticks per second).
- `sample_sizes: &'a [u32]` - Per-sample byte sizes copied into the output.
- `durations: &'a [u32]` - Per-sample durations in media timescale ticks.



## bookclerk_mp4::remux::RemuxOptions

*Struct*

Options controlling sample-copy remux (trim window, etc.).

**Fields:**
- `trim: Option<TrimRange>` - Optional start/end trim window applied during remux.

**Trait Implementations:**

- **Default**
  - `fn default() -> RemuxOptions`
- **Clone**
  - `fn clone(self: &Self) -> RemuxOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::remux::SampleTransform

*Trait*

Rewrites sample payloads as they stream from input to output.

The remuxer moves bytes and rebuilds tables; it never inspects a payload.
A caller that has to turn its own ciphertext into plaintext keeps the key and
the cipher on its side of this trait, which is how store plugins decrypt
their downloads without any of that living here. [`CopySamples`](remux.md#bookclerk_mp4remuxcopysamples) is the only
implementation in this crate.

**Methods:**

- `retain`: The original sample indices kept by the trim, in output order.
- `sample`: Rewrite output sample `index` in place. The length must not change.



## bookclerk_mp4::remux::TrimRange

*Struct*

Optional media-time trim window in milliseconds (absolute, pre-rebase).

**Fields:**
- `start_ms: u64` - Trim start offset in milliseconds from the beginning of the audio.
- `end_ms: Option<u64>` - Exclusive end; `None` means "through end of media".

**Traits:** Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TrimRange`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> TrimRange`



## bookclerk_mp4::remux::remux_progressive

*Function*

Copy (and optionally trim) a progressive MP4 into a faststart M4B, passing
every payload through `transform`.

Sample *payloads* are streamed one buffer at a time. Only the retained sizes,
durations, and read offsets are held in memory; the parsed sample table is
never cloned.

# Arguments

* `input` - Candidate score input from the public catalog.
* `output` - Filesystem path (`output`).
* `opts` - Options struct for this operation.
* `transform` - `transform` input for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn remux_progressive(input: &std::path::Path, output: &std::path::Path, opts: &RemuxOptions, transform: & mut dyn SampleTransform) -> crate::error::Result<()>
```



## bookclerk_mp4::remux::write_progressive_m4b

*Function*

Write a progressive faststart M4B by streaming one sample at a time.

Layout is written as `ftyp` + `moov` + `mdat` in a single pass (no full-file
rewrite), and `ftyp` always declares M4B brands. `fill_sample(i, buf)` must
populate `buf` with the payload for sample `i` (length must match
`sample_sizes[i]`). Only one sample buffer is live at a time.

Callers that assemble their own sample plan — from fragments, say, rather
than from one progressive `mdat` — use this directly.

# Arguments

* `output` - Filesystem path (`output`).
* `input` - Candidate score input from the public catalog.
* `fill_sample` - `fill_sample` input for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn write_progressive_m4b<F>(output: &std::path::Path, input: ProgressiveWriteInput, fill_sample: F) -> crate::error::Result<()>
```



