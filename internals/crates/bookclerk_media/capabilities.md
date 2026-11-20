# bookclerk_media::capabilities

What this build of the media worker can read and write.

Hosts ask here before handing the worker audio, instead of keeping their
own list of formats: the acquire pipeline takes a library's copy from
another store only when an output that re-encodes or repackages audio
can read it. The answers follow the crate's decode front: the
Symphonia features this crate enables, and FDK for every AAC profile.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Capabilities`](#capabilities) | struct | The codecs, containers and output encodings this build handles. |
| [`OutputEncoding`](#outputencoding) | enum | One encoding the worker can write. |
| [`decodes`](#decodes) | function | True when the worker decodes audio encoded as `codec`. |
| [`reads_content_type`](#reads_content_type) | function | True when the worker opens files of media type `content_type` (compared without case or parameters). |
| [`capabilities`](#capabilities-1) | function | The report for this build. |
| [`READABLE_CONTENT_TYPES`](#readable_content_types) | constant | Media types of whole files the worker opens: MP4 (`.m4a`/`.m4b`, AAC or ALAC), MPEG audio layers I–III, FLAC, ADTS AAC, WAV, AIFF and Core Audio Format. The container is still sniffed from the bytes when the file is opened; this list only says which kinds are worth sending. |
| [`OUTPUT_ENCODINGS`](#output_encodings) | constant | What the worker produces from decoded audio. |

## Structs

### Capabilities

```rust
pub struct Capabilities {
    pub decodes: Vec<AudioCodec>,
    pub reads: &'static [&'static str],
    pub encodes: &'static [OutputEncoding],
}
```

The codecs, containers and output encodings this build handles.

**Fields**

- `decodes`: Audio encodings the worker decodes, in [`AudioCodec`](../bookclerk_tags/README.md#audiocodec) order.
- `reads`: Media types of files the worker opens ([`READABLE_CONTENT_TYPES`](#readable_content_types)).
- `encodes`: Encodings the worker writes ([`OUTPUT_ENCODINGS`](#output_encodings)).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### OutputEncoding

```rust
pub enum OutputEncoding {
    AacLcM4b,
    Mp3,
}
```

One encoding the worker can write.

**Variants**

- `AacLcM4b`: AAC-LC in an M4B, through FDK.
- `Mp3`: MP3, through LAME.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### decodes

```rust
pub fn decodes(codec: AudioCodec) -> bool
```

True when the worker decodes audio encoded as `codec`.

AAC covers every profile (LC, HE, HE v2, xHE). MP3 covers MPEG audio
layers I and II too, which the tag reader reports as MP3.

### reads_content_type

```rust
pub fn reads_content_type(content_type: &str) -> bool
```

True when the worker opens files of media type `content_type` (compared
without case or parameters).

### capabilities

```rust
pub fn capabilities() -> Capabilities
```

The report for this build.

## Constants

### READABLE_CONTENT_TYPES

```rust
pub const READABLE_CONTENT_TYPES: &[&str] = _;
```

Media types of whole files the worker opens: MP4 (`.m4a`/`.m4b`, AAC
or ALAC), MPEG audio layers I–III, FLAC, ADTS AAC, WAV, AIFF and Core
Audio Format. The container is still sniffed from the bytes when the
file is opened; this list only says which kinds are worth sending.

### OUTPUT_ENCODINGS

```rust
pub const OUTPUT_ENCODINGS: &[OutputEncoding] = _;
```

What the worker produces from decoded audio.
