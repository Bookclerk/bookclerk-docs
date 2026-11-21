# bookclerk_tags::fixtures

Small, valid files for tests, seeds and examples: one tagged M4B, MP3 and
FLAC, built in memory. Enabled by the `fixtures` feature; not part of
the reader.

| Item | Kind | Summary |
| --- | --- | --- |
| [`mp4_box`](#mp4_box) | function | A box of `kind` around `body`. |
| [`tagged_m4b`](#tagged_m4b) | function | An M4B: ftyp, an mdat with fake audio and two chapter samples, then moov with an audio track, a text chapter track, chpl and ilst. |
| [`tagged_m4b_with_audio_config`](#tagged_m4b_with_audio_config) | function | `tagged_m4b` whose `mp4a` sample entry carries `asc` as its MPEG-4 `AudioSpecificConfig` (stereo, 44.1 kHz in the entry itself). |
| [`tagged_m4b_with_sample_entry`](#tagged_m4b_with_sample_entry) | function | `tagged_m4b` whose audio sample entry is of type `kind` (stereo, 44.1 kHz in the entry itself) with `child` as its one child box. |
| [`tagged_mp3`](#tagged_mp3) | function | A tagged MP3: an ID3v2.4 tag and a few silent frames. |
| [`vorbis_comments`](#vorbis_comments) | function | A Vorbis comment block body (vendor `vendor`) holding `items`. |
| [`tagged_flac`](#tagged_flac) | function | A FLAC file: 10 s at 44.1 kHz, three comments, two chapters, a cover. |
| [`ogg_page`](#ogg_page) | function | One Ogg page of stream `serial` with header flags `flags` and granule position `granule`, holding `packets` laced in order. A packet whose length is a multiple of 255 ends with a zero lacing value unless it is the page's last and exactly fills the table (so it continues). |
| [`tagged_ogg_opus`](#tagged_ogg_opus) | function | An Ogg Opus file: 10 s of (silent, empty) audio past a 312-sample pre-skip, a title, two chapters and a PNG cover as a `METADATA_BLOCK_PICTURE` comment. |

## Functions

### mp4_box

```rust
pub fn mp4_box(kind: &[u8; 4], body: &[u8]) -> Vec<u8>
```

A box of `kind` around `body`.

#### Panics

Never: the fixed sizes all fit their fields.

### tagged_m4b

```rust
pub fn tagged_m4b() -> Vec<u8>
```

An M4B: ftyp, an mdat with fake audio and two chapter samples, then
moov with an audio track, a text chapter track, chpl and ilst.

#### Panics

Never: the fixed sizes all fit their fields.

### tagged_m4b_with_audio_config

```rust
pub fn tagged_m4b_with_audio_config(asc: &[u8]) -> Vec<u8>
```

[`tagged_m4b`](#tagged_m4b) whose `mp4a` sample entry carries `asc` as its MPEG-4
`AudioSpecificConfig` (stereo, 44.1 kHz in the entry itself).

#### Panics

When `asc` is longer than 100 bytes, which a one-byte descriptor length
cannot hold here.

### tagged_m4b_with_sample_entry

```rust
pub fn tagged_m4b_with_sample_entry(kind: &[u8; 4], child: &[u8]) -> Vec<u8>
```

[`tagged_m4b`](#tagged_m4b) whose audio sample entry is of type `kind` (stereo,
44.1 kHz in the entry itself) with `child` as its one child box.

#### Panics

Never: the fixed sizes all fit their fields.

### tagged_mp3

```rust
pub fn tagged_mp3() -> Vec<u8>
```

A tagged MP3: an ID3v2.4 tag and a few silent frames.

#### Panics

Never: the fixed tag always encodes.

### vorbis_comments

```rust
pub fn vorbis_comments(items: &[&str]) -> Vec<u8>
```

A Vorbis comment block body (vendor `vendor`) holding `items`.

#### Panics

When an item is longer than `u32::MAX` bytes.

### tagged_flac

```rust
pub fn tagged_flac() -> Vec<u8>
```

A FLAC file: 10 s at 44.1 kHz, three comments, two chapters, a cover.

#### Panics

Never: the fixed sizes all fit their fields.

### ogg_page

```rust
pub fn ogg_page(flags: u8, granule: u64, serial: u32, packets: &[&[u8]]) -> Vec<u8>
```

One Ogg page of stream `serial` with header flags `flags` and granule
position `granule`, holding `packets` laced in order. A packet whose
length is a multiple of 255 ends with a zero lacing value unless it is
the page's last and exactly fills the table (so it continues).

#### Panics

When the packets need more than 255 lacing values.

### tagged_ogg_opus

```rust
pub fn tagged_ogg_opus() -> Vec<u8>
```

An Ogg Opus file: 10 s of (silent, empty) audio past a 312-sample
pre-skip, a title, two chapters and a PNG cover as a
`METADATA_BLOCK_PICTURE` comment.

#### Panics

Never: the fixed sizes all fit their fields.
