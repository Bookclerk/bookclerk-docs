**bookclerk_mp4 > boxutil**

# Module: boxutil

## Contents

**Structs**

- [`BoxHeader`](#boxheader) - Header for one ISO-BMFF box.
- [`FourCC`](#fourcc) - ISO-BMFF four-character type code (FourCC).

**Functions**

- [`ensure_table_entries`](#ensure_table_entries) - Rejects `count` when it exceeds [`MAX_MP4_TABLE_ENTRIES`](boxutil.md#bookclerk_mp4boxutilmax_mp4_table_entries) or when
- [`find_child`](#find_child) - Finds the first direct child box with FourCC `want` inside parent content.
- [`read_array`](#read_array) - Reads exactly `N` bytes into a stack array via `Read::read_exact`.
- [`read_box_header`](#read_box_header) - Reads an ISO-BMFF box header (size + type, including extended size).
- [`read_exact_vec`](#read_exact_vec) - Reads exactly `n` bytes into a new buffer (for dynamically sized fields).
- [`read_fourcc`](#read_fourcc) - Reads a four-byte FourCC from `data` at `offset` and advances it.
- [`read_full_box_version_flags`](#read_full_box_version_flags) - Reads a FullBox version (`u8`) and flags (`u24`) at `offset`.
- [`read_u32`](#read_u32) - Reads a big-endian `u32` from `data` at `offset` and advances it.
- [`read_u64`](#read_u64) - Reads a big-endian `u64` from `data` at `offset` and advances it.
- [`read_u8`](#read_u8) - Reads one unsigned byte from `data` at `offset` and advances it.
- [`walk_children`](#walk_children) - Iterate immediate children of a container box whose content starts at `start`

**Constants**

- [`AAVD`](#aavd) - Audible `aavd` encrypted-audio sample-entry FourCC.
- [`CO64`](#co64) - ISO-BMFF `co64` (64-bit chunk offsets) FourCC.
- [`DASH`](#dash) - ISO-BMFF `dash` brand FourCC.
- [`ENCA`](#enca) - Encrypted audio (`enca`) sample-entry FourCC.
- [`FTYP`](#ftyp) - ISO-BMFF `ftyp` (file type) FourCC.
- [`HDLR`](#hdlr) - ISO-BMFF `hdlr` (handler reference) FourCC.
- [`MAX_MP4_TABLE_ENTRIES`](#max_mp4_table_entries) - Absolute ceiling on sample / entry table lengths read from untrusted boxes.
- [`MAX_READ_EXACT_VEC_BYTES`](#max_read_exact_vec_bytes) - Maximum bytes [`read_exact_vec`](boxutil.md#bookclerk_mp4boxutilread_exact_vec) will allocate for one dynamically sized field.
- [`MDAT`](#mdat) - ISO-BMFF `mdat` (media data) FourCC.
- [`MDHD`](#mdhd) - ISO-BMFF `mdhd` (media header) FourCC.
- [`MDIA`](#mdia) - ISO-BMFF `mdia` (media) FourCC.
- [`MINF`](#minf) - ISO-BMFF `minf` (media information) FourCC.
- [`MOOF`](#moof) - ISO-BMFF `moof` (movie fragment) FourCC.
- [`MOOV`](#moov) - ISO-BMFF `moov` (movie metadata) FourCC.
- [`MP4A`](#mp4a) - MPEG-4 audio (`mp4a`) sample-entry FourCC.
- [`MVEX`](#mvex) - ISO-BMFF `mvex` (movie extends) FourCC.
- [`MVHD`](#mvhd) - ISO-BMFF `mvhd` (movie header) FourCC.
- [`SAIO`](#saio) - ISO-BMFF `saio` (sample auxiliary info offsets) FourCC.
- [`SAIZ`](#saiz) - ISO-BMFF `saiz` (sample auxiliary info sizes) FourCC.
- [`SCHI`](#schi) - ISO-BMFF `schi` (scheme information) FourCC.
- [`SCHM`](#schm) - ISO-BMFF `schm` (scheme type) FourCC.
- [`SENC`](#senc) - ISO-BMFF `senc` (sample encryption) FourCC.
- [`SIDX`](#sidx) - ISO-BMFF `sidx` (segment index) FourCC.
- [`SINF`](#sinf) - ISO-BMFF `sinf` (protection scheme info) FourCC.
- [`STBL`](#stbl) - ISO-BMFF `stbl` (sample table) FourCC.
- [`STCO`](#stco) - ISO-BMFF `stco` (32-bit chunk offsets) FourCC.
- [`STSC`](#stsc) - ISO-BMFF `stsc` (sample-to-chunk) FourCC.
- [`STSD`](#stsd) - ISO-BMFF `stsd` (sample description) FourCC.
- [`STSZ`](#stsz) - ISO-BMFF `stsz` (sample sizes) FourCC.
- [`STTS`](#stts) - ISO-BMFF `stts` (time-to-sample) FourCC.
- [`STZ2`](#stz2) - ISO-BMFF `stz2` (compact sample sizes) FourCC.
- [`TENC`](#tenc) - ISO-BMFF `tenc` (track encryption) FourCC.
- [`TFHD`](#tfhd) - ISO-BMFF `tfhd` (track fragment header) FourCC.
- [`TRAF`](#traf) - ISO-BMFF `traf` (track fragment) FourCC.
- [`TRAK`](#trak) - ISO-BMFF `trak` (track) FourCC.
- [`TRUN`](#trun) - ISO-BMFF `trun` (track fragment run) FourCC.

---

## bookclerk_mp4::boxutil::AAVD

*Constant*: `FourCC`

Audible `aavd` encrypted-audio sample-entry FourCC.



## bookclerk_mp4::boxutil::BoxHeader

*Struct*

Header for one ISO-BMFF box.

**Fields:**
- `start: u64` - Byte offset of this box or region within the file.
- `size: u64` - Total box size in bytes including the header.
- `header_len: u64` - Header length in bytes (8 or 16 for extended-size boxes).
- `kind: FourCC` - Discriminant or category for this value.

**Methods:**

- `fn content_start(self: &Self) -> u64` - Byte offset where this box's content (after the header) begins.
- `fn content_len(self: &Self) -> u64` - Payload length in bytes (total size minus header length).
- `fn end(self: &Self) -> u64` - Byte offset immediately after this box (exclusive end).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BoxHeader`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::boxutil::CO64

*Constant*: `FourCC`

ISO-BMFF `co64` (64-bit chunk offsets) FourCC.



## bookclerk_mp4::boxutil::DASH

*Constant*: `FourCC`

ISO-BMFF `dash` brand FourCC.



## bookclerk_mp4::boxutil::ENCA

*Constant*: `FourCC`

Encrypted audio (`enca`) sample-entry FourCC.



## bookclerk_mp4::boxutil::FTYP

*Constant*: `FourCC`

ISO-BMFF `ftyp` (file type) FourCC.



## bookclerk_mp4::boxutil::FourCC

*Struct*

ISO-BMFF four-character type code (FourCC).

**Tuple Struct**: `([u8; 4])`

**Methods:**

- `fn new(b: &[u8; 4]) -> Self` - Wraps a four-byte type code (no allocation).
- `fn as_str(self: &Self) -> &str` - Returns the FourCC as a UTF-8 string when the bytes are valid ASCII.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **PartialEq**
  - `fn eq(self: &Self, other: &FourCC) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> FourCC`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_mp4::boxutil::HDLR

*Constant*: `FourCC`

ISO-BMFF `hdlr` (handler reference) FourCC.



## bookclerk_mp4::boxutil::MAX_MP4_TABLE_ENTRIES

*Constant*: `usize`

Absolute ceiling on sample / entry table lengths read from untrusted boxes.

Larger than any legitimate audiobook sample table, small enough to reject
`u32::MAX`-sized allocation bombs before `Vec::with_capacity`.



## bookclerk_mp4::boxutil::MAX_READ_EXACT_VEC_BYTES

*Constant*: `usize`

Maximum bytes [`read_exact_vec`](boxutil.md#bookclerk_mp4boxutilread_exact_vec) will allocate for one dynamically sized field.

Caps DoS from a forged length while still allowing large codec configs / IVs
and modest payload slices inside a box.



## bookclerk_mp4::boxutil::MDAT

*Constant*: `FourCC`

ISO-BMFF `mdat` (media data) FourCC.



## bookclerk_mp4::boxutil::MDHD

*Constant*: `FourCC`

ISO-BMFF `mdhd` (media header) FourCC.



## bookclerk_mp4::boxutil::MDIA

*Constant*: `FourCC`

ISO-BMFF `mdia` (media) FourCC.



## bookclerk_mp4::boxutil::MINF

*Constant*: `FourCC`

ISO-BMFF `minf` (media information) FourCC.



## bookclerk_mp4::boxutil::MOOF

*Constant*: `FourCC`

ISO-BMFF `moof` (movie fragment) FourCC.



## bookclerk_mp4::boxutil::MOOV

*Constant*: `FourCC`

ISO-BMFF `moov` (movie metadata) FourCC.



## bookclerk_mp4::boxutil::MP4A

*Constant*: `FourCC`

MPEG-4 audio (`mp4a`) sample-entry FourCC.



## bookclerk_mp4::boxutil::MVEX

*Constant*: `FourCC`

ISO-BMFF `mvex` (movie extends) FourCC.



## bookclerk_mp4::boxutil::MVHD

*Constant*: `FourCC`

ISO-BMFF `mvhd` (movie header) FourCC.



## bookclerk_mp4::boxutil::SAIO

*Constant*: `FourCC`

ISO-BMFF `saio` (sample auxiliary info offsets) FourCC.



## bookclerk_mp4::boxutil::SAIZ

*Constant*: `FourCC`

ISO-BMFF `saiz` (sample auxiliary info sizes) FourCC.



## bookclerk_mp4::boxutil::SCHI

*Constant*: `FourCC`

ISO-BMFF `schi` (scheme information) FourCC.



## bookclerk_mp4::boxutil::SCHM

*Constant*: `FourCC`

ISO-BMFF `schm` (scheme type) FourCC.



## bookclerk_mp4::boxutil::SENC

*Constant*: `FourCC`

ISO-BMFF `senc` (sample encryption) FourCC.



## bookclerk_mp4::boxutil::SIDX

*Constant*: `FourCC`

ISO-BMFF `sidx` (segment index) FourCC.



## bookclerk_mp4::boxutil::SINF

*Constant*: `FourCC`

ISO-BMFF `sinf` (protection scheme info) FourCC.



## bookclerk_mp4::boxutil::STBL

*Constant*: `FourCC`

ISO-BMFF `stbl` (sample table) FourCC.



## bookclerk_mp4::boxutil::STCO

*Constant*: `FourCC`

ISO-BMFF `stco` (32-bit chunk offsets) FourCC.



## bookclerk_mp4::boxutil::STSC

*Constant*: `FourCC`

ISO-BMFF `stsc` (sample-to-chunk) FourCC.



## bookclerk_mp4::boxutil::STSD

*Constant*: `FourCC`

ISO-BMFF `stsd` (sample description) FourCC.



## bookclerk_mp4::boxutil::STSZ

*Constant*: `FourCC`

ISO-BMFF `stsz` (sample sizes) FourCC.



## bookclerk_mp4::boxutil::STTS

*Constant*: `FourCC`

ISO-BMFF `stts` (time-to-sample) FourCC.



## bookclerk_mp4::boxutil::STZ2

*Constant*: `FourCC`

ISO-BMFF `stz2` (compact sample sizes) FourCC.



## bookclerk_mp4::boxutil::TENC

*Constant*: `FourCC`

ISO-BMFF `tenc` (track encryption) FourCC.



## bookclerk_mp4::boxutil::TFHD

*Constant*: `FourCC`

ISO-BMFF `tfhd` (track fragment header) FourCC.



## bookclerk_mp4::boxutil::TRAF

*Constant*: `FourCC`

ISO-BMFF `traf` (track fragment) FourCC.



## bookclerk_mp4::boxutil::TRAK

*Constant*: `FourCC`

ISO-BMFF `trak` (track) FourCC.



## bookclerk_mp4::boxutil::TRUN

*Constant*: `FourCC`

ISO-BMFF `trun` (track fragment run) FourCC.



## bookclerk_mp4::boxutil::ensure_table_entries

*Function*

Rejects `count` when it exceeds [`MAX_MP4_TABLE_ENTRIES`](boxutil.md#bookclerk_mp4boxutilmax_mp4_table_entries) or when
`count * entry_bytes` cannot fit in `available` remaining box content.

# Arguments

* `count` - Declared entry/sample count from the box.
* `entry_bytes` - Fixed on-disk size of one table row (at least 1).
* `available` - Bytes remaining in the box after headers already consumed.
* `what` - Box or field name for error messages (e.g. `"stts"`).

# Returns

`count` as `usize` when the table is plausible.

# Errors

Returns `Mp4Error::Container` when the count is absurd or cannot fit.

```rust
fn ensure_table_entries(count: u32, entry_bytes: usize, available: u64, what: &str) -> crate::error::Result<usize>
```



## bookclerk_mp4::boxutil::find_child

*Function*

Finds the first direct child box with FourCC `want` inside parent content.

# Arguments

* `r` - `r` input for this call.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `want` - FourCC to search for among child boxes.

# Returns

On success, the inner `Option<BoxHeader>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn find_child<R>(r: & mut R, start: u64, end: u64, want: FourCC) -> crate::error::Result<Option<BoxHeader>>
```



## bookclerk_mp4::boxutil::read_array

*Function*

Reads exactly `N` bytes into a stack array via `Read::read_exact`.

# Arguments

* `r` - Reader to pull bytes from.

# Returns

An `[u8; N]` filled from `r`.

# Errors

Returns an error when the underlying I/O fails or fewer than `N` bytes are available.

```rust
fn read_array<const N, impl Read>(r: & mut impl Trait) -> crate::error::Result<[u8; N]>
```



## bookclerk_mp4::boxutil::read_box_header

*Function*

Reads an ISO-BMFF box header (size + type, including extended size).

# Arguments

* `r` - `r` input for this call.

# Returns

On success, the inner `BoxHeader` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn read_box_header<impl Read + Seek>(r: & mut impl Trait) -> crate::error::Result<BoxHeader>
```



## bookclerk_mp4::boxutil::read_exact_vec

*Function*

Reads exactly `n` bytes into a new buffer (for dynamically sized fields).

# Arguments

* `r` - Reader to pull bytes from.
* `n` - Exact number of bytes required.

# Returns

A `Vec` of length `n` filled from `r`.

# Errors

Returns an error when `n` exceeds [`MAX_READ_EXACT_VEC_BYTES`](boxutil.md#bookclerk_mp4boxutilmax_read_exact_vec_bytes), the underlying
I/O fails, or fewer than `n` bytes are available.

```rust
fn read_exact_vec<impl Read>(r: & mut impl Trait, n: usize) -> crate::error::Result<Vec<u8>>
```



## bookclerk_mp4::boxutil::read_fourcc

*Function*

Reads a four-byte FourCC from `data` at `offset` and advances it.

# Arguments

* `r` - `r` input for this call.

# Returns

On success, the inner `FourCC` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn read_fourcc<impl Read>(r: & mut impl Trait) -> crate::error::Result<FourCC>
```



## bookclerk_mp4::boxutil::read_full_box_version_flags

*Function*

Reads a FullBox version (`u8`) and flags (`u24`) at `offset`.

# Arguments

* `r` - `r` input for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn read_full_box_version_flags<impl Read>(r: & mut impl Trait) -> crate::error::Result<(u8, u32)>
```



## bookclerk_mp4::boxutil::read_u32

*Function*

Reads a big-endian `u32` from `data` at `offset` and advances it.

# Arguments

* `r` - `r` input for this call.

# Returns

On success, the inner `u32` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn read_u32<impl Read>(r: & mut impl Trait) -> crate::error::Result<u32>
```



## bookclerk_mp4::boxutil::read_u64

*Function*

Reads a big-endian `u64` from `data` at `offset` and advances it.

# Arguments

* `r` - `r` input for this call.

# Returns

On success, the inner `u64` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn read_u64<impl Read>(r: & mut impl Trait) -> crate::error::Result<u64>
```



## bookclerk_mp4::boxutil::read_u8

*Function*

Reads one unsigned byte from `data` at `offset` and advances it.

# Arguments

* `r` - `r` input for this call.

# Returns

On success, the inner `u8` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn read_u8<impl Read>(r: & mut impl Trait) -> crate::error::Result<u8>
```



## bookclerk_mp4::boxutil::walk_children

*Function*

Iterate immediate children of a container box whose content starts at `start`
and ends at `end`.

The visitor keeps its own error type, so a caller reading boxes this crate
knows nothing about can fail in its own vocabulary.

# Arguments

* `r` - `r` input for this call.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `visit` - `visit` input for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn walk_children<R, F, E>(r: & mut R, start: u64, end: u64, visit: F) -> std::result::Result<(), E>
```



