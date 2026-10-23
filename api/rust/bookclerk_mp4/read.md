**bookclerk_mp4 > read**

# Module: read

## Contents

**Structs**

- [`SampleReader`](#samplereader) - Reads sample payloads by absolute file offset, buffered.

---

## bookclerk_mp4::read::SampleReader

*Struct*

Reads sample payloads by absolute file offset, buffered.

Samples sit in track order in a progressive `mdat`, so consecutive reads are
usually already in place, and otherwise a short hop ahead. Both cases are
served out of the buffer; only a jump past it costs a real seek. Reading in
some other order still works, just without the benefit.

**Methods:**

- `fn open(path: &Path) -> Result<Self>` - Opens `path` for buffered sample reads.
- `fn new(file: File) -> Self` - Wraps an already-opened file with a buffered sample reader.
- `fn read_sample(self: & mut Self, offset: u64, size: usize, buf: & mut Vec<u8>) -> Result<()>` - Fill `buf` with exactly `size` bytes from `offset`, resizing it to match.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



