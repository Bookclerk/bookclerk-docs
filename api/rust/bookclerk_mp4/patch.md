**bookclerk_mp4 > patch**

# Module: patch

## Contents

**Structs**

- [`MoovLocation`](#moovlocation) - Where a file's `moov` sits.

**Functions**

- [`pad_moov_to`](#pad_moov_to) - Grow `moov` to exactly `total` bytes by appending a `free` child.
- [`read_moov`](#read_moov) - Read a file's `moov`, and note where it sits.
- [`shift_chunk_offsets`](#shift_chunk_offsets) - Move every chunk offset in `moov` past `above` by `delta` bytes.
- [`strip_trailing_free`](#strip_trailing_free) - Drop a `moov`'s trailing `free` child, if it has one.
- [`top_level_boxes`](#top_level_boxes) - Every box at the top level, in file order.
- [`write_moov_in_place`](#write_moov_in_place) - Write `moov` back where the old one was, padding it to the same length.

**Constants**

- [`RESERVED_MOOV_SLACK`](#reserved_moov_slack) - Slack every `moov` this crate writes carries, as a trailing `free` child.

---

## bookclerk_mp4::patch::MoovLocation

*Struct*

Where a file's `moov` sits.

**Fields:**
- `start: u64` - Byte offset of this box or region within the file.
- `len: u64` - Total box length, `free` padding included.
- `is_last: bool` - Whether `moov` is the last box in the file, in which case it can grow

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &MoovLocation) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> MoovLocation`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_mp4::patch::RESERVED_MOOV_SLACK

*Constant*: `usize`

Slack every `moov` this crate writes carries, as a trailing `free` child.

A megabyte covers a full tag set, a cover, and a chapter track for a book
with several hundred chapters — the things that get added after the media is
already on disk. Against the file it protects, which is hundreds of megabytes
of audio, it costs about a thousandth; against the alternative, which is
rewriting all of that to make room, it costs nothing at all.



## bookclerk_mp4::patch::pad_moov_to

*Function*

Grow `moov` to exactly `total` bytes by appending a `free` child.

# Errors

`Mp4Error::NoRoom` when `moov` is already longer than `total`, or when the
gap is too small to express — a `free` box cannot be shorter than its own
header, so one to seven spare bytes are as unusable as none.

# Panics

Panics when an internal invariant does not hold.

```rust
fn pad_moov_to(moov: & mut Vec<u8>, total: usize) -> crate::error::Result<()>
```



## bookclerk_mp4::patch::read_moov

*Function*

Read a file's `moov`, and note where it sits.

Only the box headers and `moov` itself are read, so the cost is the header's
and not the file's.

# Arguments

* `path` - Filesystem path involved in this operation.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn read_moov(path: &std::path::Path) -> crate::error::Result<(MoovLocation, Vec<u8>)>
```



## bookclerk_mp4::patch::shift_chunk_offsets

*Function*

Move every chunk offset in `moov` past `above` by `delta` bytes.

Chunk offsets are absolute file positions, so an edit that shifts the media
has to correct them — and an edit that only *appeared* to shift it, because
the work happened in a scratch file laid out differently, has to correct them
back. Only what sits after the edit moves, which is why `above` is usually
where `moov` starts: media in front of the header stays put.

# Arguments

* `moov` - `moov` input for this call.
* `above` - Numeric `above` value for this call.
* `delta` - Numeric `delta` value for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn shift_chunk_offsets(moov: & mut [u8], above: u64, delta: i64) -> crate::error::Result<()>
```



## bookclerk_mp4::patch::strip_trailing_free

*Function*

Drop a `moov`'s trailing `free` child, if it has one.

Anything appended to `moov` goes after its last child, so the slack has to
come off before an edit and go back on after it. Returns whether one was
there to remove.

# Arguments

* `moov` - `moov` input for this call.

# Returns

On success, the inner `bool` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn strip_trailing_free(moov: & mut Vec<u8>) -> crate::error::Result<bool>
```



## bookclerk_mp4::patch::top_level_boxes

*Function*

Every box at the top level, in file order.

# Arguments

* `file` - Open file handle.

# Returns

On success, the inner `Vec<BoxHeader>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn top_level_boxes(file: & mut std::fs::File) -> crate::error::Result<Vec<crate::boxutil::BoxHeader>>
```



## bookclerk_mp4::patch::write_moov_in_place

*Function*

Write `moov` back where the old one was, padding it to the same length.

The media does not move, so every chunk offset in the file stays true.

# Errors

`Mp4Error::NoRoom` when the rebuilt `moov` does not fit the space the old
one occupied and cannot simply be appended to.

```rust
fn write_moov_in_place(path: &std::path::Path, at: MoovLocation, moov: &[u8]) -> crate::error::Result<()>
```



