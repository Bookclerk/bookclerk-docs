**bookclerk_mp4 > fixture**

# Module: fixture

## Contents

**Structs**

- [`ProgressiveFixture`](#progressivefixture) - A progressive MP4 with one AAC audio track, described by its samples.

---

## bookclerk_mp4::fixture::ProgressiveFixture

*Struct*

A progressive MP4 with one AAC audio track, described by its samples.

**Fields:**
- `timescale: u32` - Media timescale (ticks per second) of the audio track.
- `sample_duration: u32` - Duration of every sample, in timescale ticks.
- `samples: Vec<Vec<u8>>` - Payload bytes of each sample, in track order.
- `sample_entry: [u8; 4]` - `stsd` sample entry type: `mp4a` for clear AAC, `aavd` for Audible's
- `samples_per_chunk: u32` - How many samples each chunk holds. Must divide the sample count.

**Methods:**

- `fn with_generated_samples(count: usize) -> Self` - A fixture of `count` samples whose bytes are deterministic but distinct.
- `fn with_samples(self: Self, samples: Vec<Vec<u8>>) -> Self` - Replace every sample payload, keeping the timing.
- `fn with_sample_entry(self: Self, sample_entry: &[u8; 4]) -> Self` - Set the `stsd` sample entry type.
- `fn duration_ms(self: &Self) -> u64` - Duration of the whole track in milliseconds.
- `fn write(self: &Self, path: &Path) -> Result<()>` - Write the file. Payloads land in a single trailing `mdat`.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ProgressiveFixture`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



