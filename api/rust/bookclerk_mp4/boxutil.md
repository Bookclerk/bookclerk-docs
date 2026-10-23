# bookclerk_mp4::boxutil

Low-level big-endian readers and MP4 box walking.

| Item | Kind | Summary |
| --- | --- | --- |
| [`FourCC`](#fourcc) | struct | ISO-BMFF four-character type code (FourCC). |
| [`BoxHeader`](#boxheader) | struct | Header for one ISO-BMFF box. |
| [`ensure_table_entries`](#ensure_table_entries) | function | Rejects `count` when it exceeds `MAX_MP4_TABLE_ENTRIES` or when `count * entry_bytes` cannot fit in `available` remaining box content. |
| [`read_u8`](#read_u8) | function | Reads one unsigned byte from `data` at `offset` and advances it. |
| [`read_exact_vec`](#read_exact_vec) | function | Reads exactly `n` bytes into a new buffer (for dynamically sized fields). |
| [`read_array`](#read_array) | function | Reads exactly `N` bytes into a stack array via `Read::read_exact`. |
| [`read_u32`](#read_u32) | function | Reads a big-endian `u32` from `data` at `offset` and advances it. |
| [`read_u64`](#read_u64) | function | Reads a big-endian `u64` from `data` at `offset` and advances it. |
| [`read_fourcc`](#read_fourcc) | function | Reads a four-byte FourCC from `data` at `offset` and advances it. |
| [`read_box_header`](#read_box_header) | function | Reads an ISO-BMFF box header (size + type, including extended size). |
| [`walk_children`](#walk_children) | function | Iterate immediate children of a container box whose content starts at `start` and ends at `end`. |
| [`find_child`](#find_child) | function | Finds the first direct child box with FourCC `want` inside parent content. |
| [`read_full_box_version_flags`](#read_full_box_version_flags) | function | Reads a FullBox version (`u8`) and flags (`u24`) at `offset`. |
| [`FTYP`](#ftyp) | constant | ISO-BMFF `ftyp` (file type) FourCC. |
| [`MOOV`](#moov) | constant | ISO-BMFF `moov` (movie metadata) FourCC. |
| [`MDAT`](#mdat) | constant | ISO-BMFF `mdat` (media data) FourCC. |
| [`TRAK`](#trak) | constant | ISO-BMFF `trak` (track) FourCC. |
| [`MDIA`](#mdia) | constant | ISO-BMFF `mdia` (media) FourCC. |
| [`MINF`](#minf) | constant | ISO-BMFF `minf` (media information) FourCC. |
| [`STBL`](#stbl) | constant | ISO-BMFF `stbl` (sample table) FourCC. |
| [`STSD`](#stsd) | constant | ISO-BMFF `stsd` (sample description) FourCC. |
| [`STTS`](#stts) | constant | ISO-BMFF `stts` (time-to-sample) FourCC. |
| [`STSC`](#stsc) | constant | ISO-BMFF `stsc` (sample-to-chunk) FourCC. |
| [`STSZ`](#stsz) | constant | ISO-BMFF `stsz` (sample sizes) FourCC. |
| [`STZ2`](#stz2) | constant | ISO-BMFF `stz2` (compact sample sizes) FourCC. |
| [`STCO`](#stco) | constant | ISO-BMFF `stco` (32-bit chunk offsets) FourCC. |
| [`CO64`](#co64) | constant | ISO-BMFF `co64` (64-bit chunk offsets) FourCC. |
| [`MVHD`](#mvhd) | constant | ISO-BMFF `mvhd` (movie header) FourCC. |
| [`MDHD`](#mdhd) | constant | ISO-BMFF `mdhd` (media header) FourCC. |
| [`HDLR`](#hdlr) | constant | ISO-BMFF `hdlr` (handler reference) FourCC. |
| [`AAVD`](#aavd) | constant | Audible `aavd` encrypted-audio sample-entry FourCC. |
| [`MP4A`](#mp4a) | constant | MPEG-4 audio (`mp4a`) sample-entry FourCC. |
| [`ENCA`](#enca) | constant | Encrypted audio (`enca`) sample-entry FourCC. |
| [`SIDX`](#sidx) | constant | ISO-BMFF `sidx` (segment index) FourCC. |
| [`MOOF`](#moof) | constant | ISO-BMFF `moof` (movie fragment) FourCC. |
| [`TRAF`](#traf) | constant | ISO-BMFF `traf` (track fragment) FourCC. |
| [`TFHD`](#tfhd) | constant | ISO-BMFF `tfhd` (track fragment header) FourCC. |
| [`TRUN`](#trun) | constant | ISO-BMFF `trun` (track fragment run) FourCC. |
| [`SENC`](#senc) | constant | ISO-BMFF `senc` (sample encryption) FourCC. |
| [`SINF`](#sinf) | constant | ISO-BMFF `sinf` (protection scheme info) FourCC. |
| [`SCHM`](#schm) | constant | ISO-BMFF `schm` (scheme type) FourCC. |
| [`SCHI`](#schi) | constant | ISO-BMFF `schi` (scheme information) FourCC. |
| [`TENC`](#tenc) | constant | ISO-BMFF `tenc` (track encryption) FourCC. |
| [`SAIZ`](#saiz) | constant | ISO-BMFF `saiz` (sample auxiliary info sizes) FourCC. |
| [`SAIO`](#saio) | constant | ISO-BMFF `saio` (sample auxiliary info offsets) FourCC. |
| [`MVEX`](#mvex) | constant | ISO-BMFF `mvex` (movie extends) FourCC. |
| [`DASH`](#dash) | constant | ISO-BMFF `dash` brand FourCC. |
| [`MAX_MP4_TABLE_ENTRIES`](#max_mp4_table_entries) | constant | Absolute ceiling on sample / entry table lengths read from untrusted boxes. |
| [`MAX_READ_EXACT_VEC_BYTES`](#max_read_exact_vec_bytes) | constant | Maximum bytes `read_exact_vec` will allocate for one dynamically sized field. |

## Structs

### FourCC

```rust
pub struct FourCC(pub [u8; 4]);
```

ISO-BMFF four-character type code (FourCC).

#### FourCC::new

```rust
pub const fn new(b: &[u8; 4]) -> Self
```

Wraps a four-byte type code (no allocation).

##### Arguments

* `b` - Exactly four ASCII bytes (e.g. `b"moov"`).

##### Returns

A [`FourCC`](#fourcc) viewing those bytes.

#### FourCC::as_str

```rust
pub fn as_str(&self) -> &str
```

Returns the FourCC as a UTF-8 string when the bytes are valid ASCII.

##### Returns

The type code as `&str`, or `"????"` when the bytes are not UTF-8.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Hash`, `PartialEq`

### BoxHeader

```rust
pub struct BoxHeader {
    pub start: u64,
    pub size: u64,
    pub header_len: u64,
    pub kind: FourCC,
}
```

Header for one ISO-BMFF box.

**Fields**

- `start`: Byte offset of this box or region within the file.
- `size`: Total box size in bytes including the header.
- `header_len`: Header length in bytes (8 or 16 for extended-size boxes).
- `kind`: Discriminant or category for this value.

#### BoxHeader::content_start

```rust
pub fn content_start(&self) -> u64
```

Byte offset where this box's content (after the header) begins.

#### BoxHeader::content_len

```rust
pub fn content_len(&self) -> u64
```

Payload length in bytes (total size minus header length).

#### BoxHeader::end

```rust
pub fn end(&self) -> u64
```

Byte offset immediately after this box (exclusive end).

Implements: `Clone`, `Debug`

## Functions

### ensure_table_entries

```rust
pub fn ensure_table_entries(
    count: u32,
    entry_bytes: usize,
    available: u64,
    what: &str,
) -> Result<usize>
```

Rejects `count` when it exceeds [`MAX_MP4_TABLE_ENTRIES`](#max_mp4_table_entries) or when
`count * entry_bytes` cannot fit in `available` remaining box content.

#### Arguments

* `count` - Declared entry/sample count from the box.
* `entry_bytes` - Fixed on-disk size of one table row (at least 1).
* `available` - Bytes remaining in the box after headers already consumed.
* `what` - Box or field name for error messages (e.g. `"stts"`).

#### Returns

`count` as `usize` when the table is plausible.

#### Errors

Returns [`Mp4Error::Container`](index.md#mp4error) when the count is absurd or cannot fit.

### read_u8

```rust
pub fn read_u8(r: &mut impl Read) -> Result<u8>
```

Reads one unsigned byte from `data` at `offset` and advances it.

#### Arguments

* `r` - `r` input for this call.

#### Returns

On success, the inner `u8` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### read_exact_vec

```rust
pub fn read_exact_vec(r: &mut impl Read, n: usize) -> Result<Vec<u8>>
```

Reads exactly `n` bytes into a new buffer (for dynamically sized fields).

#### Arguments

* `r` - Reader to pull bytes from.
* `n` - Exact number of bytes required.

#### Returns

A `Vec` of length `n` filled from `r`.

#### Errors

Returns an error when `n` exceeds [`MAX_READ_EXACT_VEC_BYTES`](#max_read_exact_vec_bytes), the underlying
I/O fails, or fewer than `n` bytes are available.

### read_array

```rust
pub fn read_array<const N: usize>(r: &mut impl Read) -> Result<[u8; N]>
```

Reads exactly `N` bytes into a stack array via `Read::read_exact`.

#### Arguments

* `r` - Reader to pull bytes from.

#### Returns

An `[u8; N]` filled from `r`.

#### Errors

Returns an error when the underlying I/O fails or fewer than `N` bytes are available.

### read_u32

```rust
pub fn read_u32(r: &mut impl Read) -> Result<u32>
```

Reads a big-endian `u32` from `data` at `offset` and advances it.

#### Arguments

* `r` - `r` input for this call.

#### Returns

On success, the inner `u32` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### read_u64

```rust
pub fn read_u64(r: &mut impl Read) -> Result<u64>
```

Reads a big-endian `u64` from `data` at `offset` and advances it.

#### Arguments

* `r` - `r` input for this call.

#### Returns

On success, the inner `u64` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### read_fourcc

```rust
pub fn read_fourcc(r: &mut impl Read) -> Result<FourCC>
```

Reads a four-byte FourCC from `data` at `offset` and advances it.

#### Arguments

* `r` - `r` input for this call.

#### Returns

On success, the inner `FourCC` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### read_box_header

```rust
pub fn read_box_header(r: &mut impl Read + Seek) -> Result<BoxHeader>
```

Reads an ISO-BMFF box header (size + type, including extended size).

#### Arguments

* `r` - `r` input for this call.

#### Returns

On success, the inner `BoxHeader` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### walk_children

```rust
pub fn walk_children<R, F, E>(r: &mut R, start: u64, end: u64, visit: F) -> Result<(), E>
where
    R: Read + Seek,
    F: FnMut(&mut R, &BoxHeader) -> Result<(), E>,
    E: From<Mp4Error>,
```

Iterate immediate children of a container box whose content starts at `start`
and ends at `end`.

The visitor keeps its own error type, so a caller reading boxes this crate
knows nothing about can fail in its own vocabulary.

#### Arguments

* `r` - `r` input for this call.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `visit` - `visit` input for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### find_child

```rust
pub fn find_child<R: Read + Seek>(
    r: &mut R,
    start: u64,
    end: u64,
    want: FourCC,
) -> Result<Option<BoxHeader>>
```

Finds the first direct child box with FourCC `want` inside parent content.

#### Arguments

* `r` - `r` input for this call.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `want` - FourCC to search for among child boxes.

#### Returns

On success, the inner `Option<BoxHeader>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### read_full_box_version_flags

```rust
pub fn read_full_box_version_flags(r: &mut impl Read) -> Result<(u8, u32)>
```

Reads a FullBox version (`u8`) and flags (`u24`) at `offset`.

#### Arguments

* `r` - `r` input for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

## Constants

### FTYP

```rust
pub const FTYP: FourCC = _;
```

ISO-BMFF `ftyp` (file type) FourCC.

### MOOV

```rust
pub const MOOV: FourCC = _;
```

ISO-BMFF `moov` (movie metadata) FourCC.

### MDAT

```rust
pub const MDAT: FourCC = _;
```

ISO-BMFF `mdat` (media data) FourCC.

### TRAK

```rust
pub const TRAK: FourCC = _;
```

ISO-BMFF `trak` (track) FourCC.

### MDIA

```rust
pub const MDIA: FourCC = _;
```

ISO-BMFF `mdia` (media) FourCC.

### MINF

```rust
pub const MINF: FourCC = _;
```

ISO-BMFF `minf` (media information) FourCC.

### STBL

```rust
pub const STBL: FourCC = _;
```

ISO-BMFF `stbl` (sample table) FourCC.

### STSD

```rust
pub const STSD: FourCC = _;
```

ISO-BMFF `stsd` (sample description) FourCC.

### STTS

```rust
pub const STTS: FourCC = _;
```

ISO-BMFF `stts` (time-to-sample) FourCC.

### STSC

```rust
pub const STSC: FourCC = _;
```

ISO-BMFF `stsc` (sample-to-chunk) FourCC.

### STSZ

```rust
pub const STSZ: FourCC = _;
```

ISO-BMFF `stsz` (sample sizes) FourCC.

### STZ2

```rust
pub const STZ2: FourCC = _;
```

ISO-BMFF `stz2` (compact sample sizes) FourCC.

### STCO

```rust
pub const STCO: FourCC = _;
```

ISO-BMFF `stco` (32-bit chunk offsets) FourCC.

### CO64

```rust
pub const CO64: FourCC = _;
```

ISO-BMFF `co64` (64-bit chunk offsets) FourCC.

### MVHD

```rust
pub const MVHD: FourCC = _;
```

ISO-BMFF `mvhd` (movie header) FourCC.

### MDHD

```rust
pub const MDHD: FourCC = _;
```

ISO-BMFF `mdhd` (media header) FourCC.

### HDLR

```rust
pub const HDLR: FourCC = _;
```

ISO-BMFF `hdlr` (handler reference) FourCC.

### AAVD

```rust
pub const AAVD: FourCC = _;
```

Audible `aavd` encrypted-audio sample-entry FourCC.

### MP4A

```rust
pub const MP4A: FourCC = _;
```

MPEG-4 audio (`mp4a`) sample-entry FourCC.

### ENCA

```rust
pub const ENCA: FourCC = _;
```

Encrypted audio (`enca`) sample-entry FourCC.

### SIDX

```rust
pub const SIDX: FourCC = _;
```

ISO-BMFF `sidx` (segment index) FourCC.

### MOOF

```rust
pub const MOOF: FourCC = _;
```

ISO-BMFF `moof` (movie fragment) FourCC.

### TRAF

```rust
pub const TRAF: FourCC = _;
```

ISO-BMFF `traf` (track fragment) FourCC.

### TFHD

```rust
pub const TFHD: FourCC = _;
```

ISO-BMFF `tfhd` (track fragment header) FourCC.

### TRUN

```rust
pub const TRUN: FourCC = _;
```

ISO-BMFF `trun` (track fragment run) FourCC.

### SENC

```rust
pub const SENC: FourCC = _;
```

ISO-BMFF `senc` (sample encryption) FourCC.

### SINF

```rust
pub const SINF: FourCC = _;
```

ISO-BMFF `sinf` (protection scheme info) FourCC.

### SCHM

```rust
pub const SCHM: FourCC = _;
```

ISO-BMFF `schm` (scheme type) FourCC.

### SCHI

```rust
pub const SCHI: FourCC = _;
```

ISO-BMFF `schi` (scheme information) FourCC.

### TENC

```rust
pub const TENC: FourCC = _;
```

ISO-BMFF `tenc` (track encryption) FourCC.

### SAIZ

```rust
pub const SAIZ: FourCC = _;
```

ISO-BMFF `saiz` (sample auxiliary info sizes) FourCC.

### SAIO

```rust
pub const SAIO: FourCC = _;
```

ISO-BMFF `saio` (sample auxiliary info offsets) FourCC.

### MVEX

```rust
pub const MVEX: FourCC = _;
```

ISO-BMFF `mvex` (movie extends) FourCC.

### DASH

```rust
pub const DASH: FourCC = _;
```

ISO-BMFF `dash` brand FourCC.

### MAX_MP4_TABLE_ENTRIES

```rust
pub const MAX_MP4_TABLE_ENTRIES: usize = 10_000_000;
```

Absolute ceiling on sample / entry table lengths read from untrusted boxes.

Larger than any legitimate audiobook sample table, small enough to reject
`u32::MAX`-sized allocation bombs before `Vec::with_capacity`.

### MAX_READ_EXACT_VEC_BYTES

```rust
pub const MAX_READ_EXACT_VEC_BYTES: usize = _;
```

Maximum bytes [`read_exact_vec`](#read_exact_vec) will allocate for one dynamically sized field.

Caps DoS from a forged length while still allowing large codec configs / IVs
and modest payload slices inside a box.
