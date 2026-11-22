# bookclerk_libav

Decoders-only FFmpeg for the audio formats Bookclerk's own decoders do
not cover: AC-3 and E-AC-3, Windows Media Audio (v1, v2, Pro and
Lossless) and AMR (narrowband and wideband).

The build script compiles a pinned FFmpeg release with nothing but
those decoders, their demuxers (ASF, raw AC-3/E-AC-3, raw AMR and
ISO-BMFF for 3GP) and the AC-3 parser, and links it statically. A
small C shim (`src/shim.c`) is the only code that touches FFmpeg's
structs, so this crate never depends on their layout.

The media worker is the only user: it decodes untrusted files with
[`Demuxer`](#demuxer) and [`Decoder`](#decoder) inside its confinement, the same as every
other codec. FFmpeg is LGPL-2.1-or-later in this configuration; see
`docs/contributing/licensing.md`.

Unix and Windows (MSVC) only: on other targets the crate is empty, and
the `libav` feature of `bookclerk-media` that pulls it in stays off
there.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Source`](#source) | trait | A seekable byte source a `Demuxer` reads from. |
| [`CodecConfig`](#codecconfig) | struct | What a decoder needs to know about a track. |
| [`Frame`](#frame) | struct | One decoded frame: interleaved 32-bit float samples in -1.0..=1.0. |
| [`Decoder`](#decoder) | struct | An FFmpeg audio decoder for one track. |
| [`Track`](#track) | struct | The audio track a `Demuxer` chose. |
| [`Packet`](#packet) | struct | One demuxed packet of the chosen track. |
| [`Demuxer`](#demuxer) | struct | An FFmpeg demuxer over a byte source, reading one audio track. |
| [`Error`](#error) | enum | Why a call into FFmpeg failed. |
| [`Codec`](#codec) | enum | A codec this build decodes. |
| [`sniff`](#sniff) | function | Whether `head`, the first bytes of a file, starts a container that only this crate reads: ASF, raw AMR (`#!AMR`) or a raw AC-3/E-AC-3 stream (sync word `0x0B77` at offset 0). |
| [`FFMPEG_VERSION`](#ffmpeg_version) | constant | The FFmpeg release this crate builds, as `configure` names it. |
| [`MAX_EXTRADATA`](#max_extradata) | constant | Most bytes of codec setup data accepted from a container (1 MiB), matching the shim's own limit. |
| [`SNIFF_LEN`](#sniff_len) | constant | Bytes `sniff` needs to decide. |

## Modules

- [`fixtures`](fixtures.md): Small audio files in every format this crate decodes, for tests here and in the crates that use it.

## Traits

### Source

```rust
pub trait Source: Read + Seek + Send {}
```

A seekable byte source a [`Demuxer`](#demuxer) reads from.

Implemented by: `T`

## Structs

### CodecConfig

```rust
pub struct CodecConfig {
    pub codec: Codec,
    pub sample_rate: u32,
    pub channels: u16,
    pub block_align: u32,
    pub bit_rate: u64,
    pub extradata: Vec<u8>,
}
```

What a decoder needs to know about a track.

**Fields**

- `codec`: The codec.
- `sample_rate`: Stated sample rate in Hz, or 0 when the stream carries its own.
- `channels`: Stated channel count, or 0 when the stream carries its own.
- `block_align`: Bytes per coded block (WMA needs it), or 0.
- `bit_rate`: Stated bit rate in bits per second (WMA needs it), or 0.
- `extradata`: Codec setup bytes from the container (WMA needs them), at most
  [`MAX_EXTRADATA`](#max_extradata).

#### CodecConfig::bare

```rust
pub fn bare(codec: Codec) -> Self
```

A config for `codec` with nothing stated: enough for AC-3, E-AC-3
and AMR, whose streams describe themselves.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Frame

```rust
pub struct Frame<'a> {
    pub samples: &'a [f32],
    pub frames: usize,
    pub channels: usize,
    pub sample_rate: u32,
    pub channel_mask: u64,
}
```

One decoded frame: interleaved 32-bit float samples in -1.0..=1.0.

**Fields**

- `samples`: `frames * channels` samples, channel-interleaved.
- `frames`: Samples per channel.
- `channels`: Interleaved channels, 1 to 8, in FFmpeg's default order for the
  count (left, right, centre, LFE, …).
- `sample_rate`: Sample rate in Hz.
- `channel_mask`: Speaker positions in WAVE bit numbering (bit 0 front left, 1 front
  right, 2 centre, 3 LFE, …), one bit per channel in interleaved
  order; 0 when the decoder states none.

Implements: `Clone`, `Copy`, `Debug`

### Decoder

```rust
pub struct Decoder {
    /* private fields */
}
```

An FFmpeg audio decoder for one track.

#### Decoder::new

```rust
pub fn new(config: &CodecConfig) -> Result<Self, Error>
```

Opens a decoder for `config`.

##### Errors

[`Error::Unsupported`](#error) when FFmpeg will not open the codec with
these parameters, [`Error::Arguments`](#error) for out-of-range values, or
[`Error::OutOfMemory`](#error).

#### Decoder::decode

```rust
pub fn decode<F>(&mut self, packet: &[u8], on_frame: F) -> Result<(), Error>
where
    F: FnMut(Frame<'_>),
```

Decodes one packet, calling `on_frame` for each frame it gives.

##### Errors

[`Error::Decode`](#error) when the decoder rejects the packet (the caller
may skip it and go on), [`Error::Layout`](#error) for a frame outside the
shim's limits, or [`Error::OutOfMemory`](#error).

#### Decoder::drain

```rust
pub fn drain<F>(&mut self, on_frame: F) -> Result<(), Error>
where
    F: FnMut(Frame<'_>),
```

Ends the stream: hands back the frames a delayed decoder (WMA v1,
v2 and lossless) still holds, then resets it so later packets, as
after a seek, decode from a clean state.

##### Errors

[`Error::Decode`](#error) when the decoder refuses the end of stream,
[`Error::Layout`](#error) for a frame outside the shim's limits, or
[`Error::OutOfMemory`](#error). The decoder is reset either way.

#### Decoder::flush

```rust
pub fn flush(&mut self)
```

Drops buffered decoder state, as after a seek.

Implements: `Drop`, `Send`, `Sync`

### Track

```rust
pub struct Track {
    pub config: CodecConfig,
    pub time_base: (u32, u32),
    pub duration: Option<u64>,
    pub start: i64,
}
```

The audio track a [`Demuxer`](#demuxer) chose.

**Fields**

- `config`: What a [`Decoder`](#decoder) needs for it.
- `time_base`: Seconds per timestamp tick: `time_base.0 / time_base.1`, both
  positive.
- `duration`: Length in ticks, when the container states it.
- `start`: First timestamp in ticks.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Packet

```rust
pub struct Packet<'a> {
    pub data: &'a [u8],
    pub pts: i64,
    pub duration: u64,
}
```

One demuxed packet of the chosen track.

**Fields**

- `data`: The coded bytes, valid until the next demuxer call.
- `pts`: Presentation timestamp in ticks (may be negative or missing as
  `i64::MIN` for some streams).
- `duration`: Length in ticks, or 0 when not stated.

Implements: `Clone`, `Copy`, `Debug`

### Demuxer

```rust
pub struct Demuxer {
    /* private fields */
}
```

An FFmpeg demuxer over a byte source, reading one audio track.

#### Demuxer::open

```rust
pub fn open(source: Box<dyn Source>) -> Result<Self, Error>
```

Opens the container in `source` and picks its best audio track.

##### Errors

[`Error::Demux`](#error) when no enabled demuxer recognises the bytes or
the header is broken, [`Error::Unsupported`](#error) when there is no audio
track in a codec this build decodes, or [`Error::OutOfMemory`](#error).

#### Demuxer::track

```rust
pub fn track(&self) -> &Track
```

The chosen audio track.

#### Demuxer::next_packet

```rust
pub fn next_packet(&mut self) -> Result<Option<Packet<'_>>, Error>
```

Reads the chosen track's next packet.

##### Returns

The packet, or `None` at the end of the stream.

##### Errors

[`Error::Demux`](#error) when the container is broken (or the source
failed), or [`Error::OutOfMemory`](#error).

#### Demuxer::seek

```rust
pub fn seek(&mut self, ts: i64) -> Result<(), Error>
```

Seeks to the last key point at or before `ts` (in track ticks).

##### Errors

[`Error::Demux`](#error) when the container cannot seek there.

Implements: `Drop`, `Send`, `Sync`

## Enums

### Error

```rust
pub enum Error {
    Arguments,
    OutOfMemory,
    Unsupported,
    Decode,
    Demux,
    Layout,
}
```

Why a call into FFmpeg failed.

**Variants**

- `Arguments`: A value handed to the shim was out of range.
- `OutOfMemory`: An allocation failed.
- `Unsupported`: This build has no decoder or demuxer for the input.
- `Decode`: The decoder rejected a packet as corrupt.
- `Demux`: The demuxer failed.
- `Layout`: A decoded frame had a channel count, rate or length outside the
  shim's limits.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### Codec

```rust
pub enum Codec {
    Ac3,
    Eac3,
    WmaV1,
    WmaV2,
    WmaPro,
    WmaLossless,
    AmrNb,
    AmrWb,
}
```

A codec this build decodes.

**Variants**

- `Ac3`: Dolby Digital (AC-3).
- `Eac3`: Dolby Digital Plus (E-AC-3).
- `WmaV1`: Windows Media Audio 1.
- `WmaV2`: Windows Media Audio 2.
- `WmaPro`: Windows Media Audio 9 Professional.
- `WmaLossless`: Windows Media Audio 9 Lossless.
- `AmrNb`: Adaptive Multi-Rate narrowband (8 kHz speech).
- `AmrWb`: Adaptive Multi-Rate wideband (16 kHz speech).

#### Codec::ALL

```rust
pub const ALL: [Codec; 8] = _
```

Every codec, in the shim's numbering order.

#### Codec::number

```rust
pub fn number(self) -> u8
```

The shim's number for this codec (1 to 8).

#### Codec::from_number

```rust
pub fn from_number(number: u8) -> Option<Self>
```

The codec with shim number `number`, if any.

#### Codec::name

```rust
pub fn name(self) -> &'static str
```

FFmpeg's short name for the decoder (`ac3`, `wmav2`, `amrwb`, …).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `PartialEq`

## Functions

### sniff

```rust
pub fn sniff(head: &[u8]) -> bool
```

Whether `head`, the first bytes of a file, starts a container that
only this crate reads: ASF, raw AMR (`#!AMR`) or a raw AC-3/E-AC-3
stream (sync word `0x0B77` at offset 0).

Callers check this before handing the file to another demuxer, which
might otherwise mistake a raw stream for MPEG audio while hunting for
frame syncs.

## Constants

### FFMPEG_VERSION

```rust
pub const FFMPEG_VERSION: &str = "9.0.2";
```

The FFmpeg release this crate builds, as `configure` names it.

### MAX_EXTRADATA

```rust
pub const MAX_EXTRADATA: usize = _;
```

Most bytes of codec setup data accepted from a container (1 MiB),
matching the shim's own limit.

### SNIFF_LEN

```rust
pub const SNIFF_LEN: usize = 16;
```

Bytes [`sniff`](#sniff) needs to decide.
