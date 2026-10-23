# bookclerk_mp4::fixture

Synthetic progressive MP4 files for tests.

Behind the `fixtures` feature so it never reaches a release build. Tests in
this crate and in the store plugins both need real files to parse — a plugin
proving its decrypt round-trips through [`crate::remux_progressive`](index.md#remux_progressive) needs
one that carries its sample entry — and a writer they share is one fewer
copy of ISO-BMFF layout to keep correct.

## Structs

### ProgressiveFixture

```rust
pub struct ProgressiveFixture {
    pub timescale: u32,
    pub sample_duration: u32,
    pub samples: Vec<Vec<u8>>,
    pub sample_entry: [u8; 4],
    pub samples_per_chunk: u32,
}
```

A progressive MP4 with one AAC audio track, described by its samples.

**Fields**

- `timescale`: Media timescale (ticks per second) of the audio track.
- `sample_duration`: Duration of every sample, in timescale ticks.
- `samples`: Payload bytes of each sample, in track order.
- `sample_entry`: `stsd` sample entry type: `mp4a` for clear AAC, `aavd` for Audible's
  encrypted flavour, or anything else a reader should cope with.
- `samples_per_chunk`: How many samples each chunk holds. Must divide the sample count.

#### ProgressiveFixture::with_generated_samples

```rust
pub fn with_generated_samples(count: usize) -> Self
```

A fixture of `count` samples whose bytes are deterministic but distinct.

##### Arguments

* `count` - Numeric `count` value for this call.

##### Returns

Updated `Self` for chaining.

#### ProgressiveFixture::with_samples

```rust
pub fn with_samples(self, samples: Vec<Vec<u8>>) -> Self
```

Replace every sample payload, keeping the timing.

##### Arguments

* `samples` - `samples` input for this call.

##### Returns

Updated `Self` for chaining.

#### ProgressiveFixture::with_sample_entry

```rust
pub fn with_sample_entry(self, sample_entry: &[u8; 4]) -> Self
```

Set the `stsd` sample entry type.

##### Arguments

* `sample_entry` - `sample_entry` input for this call.

##### Returns

Updated `Self` for chaining.

#### ProgressiveFixture::duration_ms

```rust
pub fn duration_ms(&self) -> u64
```

Duration of the whole track in milliseconds.

#### ProgressiveFixture::write

```rust
pub fn write(&self, path: &Path) -> Result<()>
```

Write the file. Payloads land in a single trailing `mdat`.

##### Arguments

* `path` - Filesystem path involved in this operation.

##### Returns

The successful result value for this operation.

##### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

##### Panics

Panics when an internal invariant does not hold.

Implements: `Clone`, `Debug`, `Default`
