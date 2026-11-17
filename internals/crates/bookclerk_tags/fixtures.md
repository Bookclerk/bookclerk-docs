# bookclerk_tags::fixtures

Small, valid files for tests, seeds and examples: one tagged M4B, MP3 and
FLAC, built in memory. Enabled by the `fixtures` feature; not part of
the reader.

| Item | Kind | Summary |
| --- | --- | --- |
| [`mp4_box`](#mp4_box) | function | A box of `kind` around `body`. |
| [`tagged_m4b`](#tagged_m4b) | function | An M4B: ftyp, an mdat with fake audio and two chapter samples, then moov with an audio track, a text chapter track, chpl and ilst. |
| [`tagged_mp3`](#tagged_mp3) | function | A tagged MP3: an ID3v2.4 tag and a few silent frames. |
| [`tagged_flac`](#tagged_flac) | function | A FLAC file: 10 s at 44.1 kHz, three comments, two chapters, a cover. |

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

### tagged_mp3

```rust
pub fn tagged_mp3() -> Vec<u8>
```

A tagged MP3: an ID3v2.4 tag and a few silent frames.

#### Panics

Never: the fixed tag always encodes.

### tagged_flac

```rust
pub fn tagged_flac() -> Vec<u8>
```

A FLAC file: 10 s at 44.1 kHz, three comments, two chapters, a cover.

#### Panics

Never: the fixed sizes all fit their fields.
