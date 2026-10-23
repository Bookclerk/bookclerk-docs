**bookclerk_mp4 > samples**

# Module: samples

## Contents

**Structs**

- [`ChunkMapEntry`](#chunkmapentry) - One run from the sample-to-chunk (`stsc`) table.
- [`SampleInfo`](#sampleinfo) - One audio sample, located and timed.

**Functions**

- [`build_samples`](#build_samples) - Builds a flat sample table from `stts` / `stsc` / `stsz` / `stco` (or `co64`).
- [`select_samples_by_ms`](#select_samples_by_ms) - Indices of the samples whose media time overlaps `[start_ms, end_ms)`.

---

## bookclerk_mp4::samples::ChunkMapEntry

*Struct*

One run from the sample-to-chunk (`stsc`) table.

**Fields:**
- `first_chunk: u32` - 1-based first chunk index for this `stsc` entry.
- `samples_per_chunk: u32` - Number of samples in each chunk of this run.
- `sample_description_index: u32` - 1-based index into the sample description table.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ChunkMapEntry`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::samples::SampleInfo

*Struct*

One audio sample, located and timed.

**Fields:**
- `offset: u64` - Absolute file offset of the sample payload.
- `size: u32` - Total box size in bytes including the header.
- `start_cts: u64` - Composition start time in media timescale ticks.
- `duration: u32` - Sample duration in media timescale ticks.
- `chunk_index: u32` - 1-based chunk index that contains this sample.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SampleInfo`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::samples::build_samples

*Function*

Builds a flat sample table from `stts` / `stsc` / `stsz` / `stco` (or `co64`).

# Arguments

* `stts` - Numeric `stts` value for this call.
* `stsc` - `stsc` input for this call.
* `sample_sizes` - Numeric `sample_sizes` value for this call.
* `chunk_offsets` - Numeric `chunk_offsets` value for this call.

# Returns

On success, the inner `Vec<SampleInfo>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn build_samples(stts: &[(u32, u32)], stsc: &[ChunkMapEntry], sample_sizes: &[u32], chunk_offsets: &[u64]) -> crate::error::Result<Vec<SampleInfo>>
```



## bookclerk_mp4::samples::select_samples_by_ms

*Function*

Indices of the samples whose media time overlaps `[start_ms, end_ms)`.

Indices rather than clones: a long audiobook has millions of samples, and the
caller needs the original positions anyway to line up any per-sample state it
keeps of its own. Output composition times are rebased by the writer.

# Arguments

* `samples` - `samples` input for this call.
* `timescale` - Numeric `timescale` value for this call.
* `start_ms` - Numeric `start_ms` value for this call.
* `end_ms` - Numeric `end_ms` value for this call.

# Returns

Collected results (may be empty).

```rust
fn select_samples_by_ms(samples: &[SampleInfo], timescale: u32, start_ms: u64, end_ms: Option<u64>) -> Vec<usize>
```



