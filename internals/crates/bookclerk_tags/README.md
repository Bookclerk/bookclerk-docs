# bookclerk_tags

Reads the metadata embedded in audiobook files without reading the audio.

[`TagReader`](#tagreader) is a sans-IO state machine: it never opens a file. It asks
for byte ranges ([`Poll::Need`](#poll)), the caller supplies them
([`TagReader::supply`](#tagreadersupply)), and once it has the bytes that hold the tags it
answers with a [`TagRecord`]: text entries, chapters, the running time,
where the cover picture lies, and a digest of the metadata region. The
same code serves a local file, an S3 object read by ranged `GET`, the
fuzzer and the unit tests, and only the metadata region (plus a few box
headers) is ever fetched.

Files on storage are untrusted. The reader runs inside
`bookclerk-media-worker tags`, a long-lived worker jailed to no path and
no network, which speaks [`wire`](wire.md) frames with the host. The host checks
every record it gets back with [`TagRecord::validate`](#tagrecordvalidate).

Supported containers: MP4 (`ilst` items, QuickTime chapter tracks and
Nero `chpl`), MP3 (ID3v2 with `CHAP`, ID3v1, Xing/Info/VBRI frame counts)
FLAC (Vorbis comments, `PICTURE`, `STREAMINFO`) and Ogg Opus or Vorbis
(comment header, `METADATA_BLOCK_PICTURE`, last granule position).

| Item | Kind | Summary |
| --- | --- | --- |
| [`TagReader`](#tagreader) | struct | Reads one file's tags, asking for the bytes it needs. |
| [`AudioFormat`](#audioformat) | struct | What a stream's headers state about its audio, besides the codec. |
| [`Chapter`](#chapter) | struct | One chapter mark. |
| [`Duration`](#duration) | struct | A running time and how it was found. |
| [`EmbeddedCover`](#embeddedcover) | struct | Where the embedded cover is and what it is, without its bytes. |
| [`RegionDigest`](#regiondigest) | struct | Digest of a file's metadata region. |
| [`Sha256Digest`](#sha256digest) | struct | A SHA-256 digest, written as 64 lowercase hex digits. |
| [`TagEntry`](#tagentry) | struct | One text value from the file. |
| [`TagRecord`](#tagrecord) | struct | The metadata one file carries in its own container. |
| [`ByteSpan`](#bytespan) | struct | A range of bytes in a file. |
| [`Poll`](#poll) | enum | What `TagReader::poll` wants next. |
| [`AacProfile`](#aacprofile) | enum | The AAC profiles a library tells apart, from the audio object type of an MPEG-4 `AudioSpecificConfig`. |
| [`AudioCodec`](#audiocodec) | enum | The audio encodings the reader tells apart. |
| [`ChapterSource`](#chaptersource) | enum | Which structure a file's chapters were read from. |
| [`Container`](#container) | enum | The container formats the reader understands. |
| [`DurationPrecision`](#durationprecision) | enum | How a `Duration` was found. |
| [`PictureFormat`](#pictureformat) | enum | The format of an embedded picture, from its signature. |
| [`RecordError`](#recorderror) | enum | Why a `TagRecord` from the worker was refused. |
| [`TagField`](#tagfield) | enum | The field a `TagEntry` fills. |
| [`TagError`](#tagerror) | enum | Why a file's tags could not be read. |
| [`read_all`](#read_all) | function | Reads a whole in-memory file in one pass. |
| [`MAX_CHANNELS`](#max_channels) | constant | Most channels a record may claim: above any layout an audiobook or a surround mix uses, so a larger value means a corrupt header. |
| [`MAX_DURATION_MS`](#max_duration_ms) | constant | Longest running time a record may claim: 10,000 hours, in milliseconds. |
| [`MAX_SAMPLE_RATE_HZ`](#max_sample_rate_hz) | constant | Highest sample rate a record may claim, in hertz (768 kHz, the top of any PCM or FLAC stream). |
| [`REGION_HASH_VERSION`](#region_hash_version) | constant | Version of the region digest this reader computes (`RegionDigest::version`). |
| [`MAX_ENTRIES`](#max_entries) | constant | Most text entries a record carries; the rest are dropped and the record marked truncated. |
| [`MAX_VALUE_BYTES`](#max_value_bytes) | constant | Longest value an entry carries, in bytes; longer values are cut at a character boundary and the record marked truncated. |
| [`MAX_KEY_BYTES`](#max_key_bytes) | constant | Longest key (freeform name, namespace or native id), in bytes; an entry with a longer key is dropped. |
| [`MAX_CHAPTERS`](#max_chapters) | constant | Most chapters a record carries. |
| [`MAX_CHAPTER_TITLE_BYTES`](#max_chapter_title_bytes) | constant | Longest chapter title, in bytes. |
| [`MAX_FETCH_BYTES`](#max_fetch_bytes) | constant | Most bytes the reader asks for over one file. Real metadata, cover included, is a few megabytes at most. |
| [`MAX_ROUNDS`](#max_rounds) | constant | Most rounds of `Poll::Need` for one file. |
| [`MAX_SPANS_PER_ROUND`](#max_spans_per_round) | constant | Most spans one `Poll::Need` asks for. |
| [`MIN_FETCH_BYTES`](#min_fetch_bytes) | constant | Least the reader asks for at once when it only needs a header, so a walk over small boxes takes few rounds. |

## Modules

- [`fixtures`](fixtures.md): Small, valid files for tests, seeds and examples: one tagged M4B, MP3 and FLAC, built in memory. Enabled by the `fixtures` feature; not part of the reader.
- [`wire`](wire.md): Frames the host and `bookclerk-media-worker tags` exchange over the worker's stdio.

## Structs

### TagReader

```rust
pub struct TagReader {
    /* private fields */
}
```

Reads one file's tags, asking for the bytes it needs.

Each poll runs the parser from the start over the bytes supplied so far.
A pass that reaches bytes it does not have stops and asks for them (all
the ones it can name at once), so the number of rounds is about the
depth of the container, not the number of fields. Headers are fetched
in windows of at least [`MIN_FETCH_BYTES`](#min_fetch_bytes).

#### TagReader::new

```rust
pub fn new(size: u64) -> Self
```

A reader for a file of `size` bytes.

#### TagReader::poll

```rust
pub fn poll(&mut self) -> Poll
```

Advances the reader.

##### Returns

The spans to supply next, or the answer. While spans from the last
[`Poll::Need`](#poll) are still outstanding it asks for those again.

#### TagReader::is_waiting

```rust
pub fn is_waiting(&self) -> bool
```

Whether spans from the last [`Poll::Need`](#poll) are still outstanding, so
polling now would only ask for them again.

#### TagReader::supply

```rust
pub fn supply(&mut self, offset: u64, bytes: Vec<u8>) -> Result<(), TagError>
```

Supplies the bytes of one span the last [`Poll::Need`](#poll) asked for.

##### Arguments

* `offset` - The span's offset.
* `bytes` - Exactly the span's bytes.

##### Errors

[`TagError::Unrequested`](#tagerror) when no outstanding span starts at
`offset` with that length; the reader is then done with that error.

Implements: `Debug`

### AudioFormat

```rust
pub struct AudioFormat {
    pub channels: Option<u8>,
    pub sample_rate_hz: Option<u32>,
    pub aac_profile: Option<AacProfile>,
}
```

What a stream's headers state about its audio, besides the codec.

Each value is `None` when the headers do not state it. Nothing is
decoded to find these, so an AAC stream that signals HE-AAC only
implicitly (in its first frames) reads as [`AacProfile::Lc`](#aacprofile).

**Fields**

- `channels`: Output channels: 1 for mono, 2 for stereo; at most [`MAX_CHANNELS`](#max_channels).
- `sample_rate_hz`: Output sample rate in hertz (for HE-AAC, the rate after spectral
  band replication; for Opus, the encoder's input rate when the stream
  states it); at most [`MAX_SAMPLE_RATE_HZ`](#max_sample_rate_hz).
- `aac_profile`: The AAC profile, for an AAC stream whose decoder configuration names
  one the reader knows.

#### AudioFormat::is_unknown

```rust
pub fn is_unknown(&self) -> bool
```

True when the headers stated none of the values.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Chapter

```rust
pub struct Chapter {
    pub start_ms: u64,
    pub title: String,
}
```

One chapter mark.

**Fields**

- `start_ms`: Start, in milliseconds from the beginning of the file.
- `title`: Title; at most [`MAX_CHAPTER_TITLE_BYTES`](#max_chapter_title_bytes) bytes, never containing NUL.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Duration

```rust
pub struct Duration {
    pub ms: u64,
    pub precision: DurationPrecision,
}
```

A running time and how it was found.

**Fields**

- `ms`: Milliseconds; at most [`MAX_DURATION_MS`](#max_duration_ms).
- `precision`: Whether the container states it or it was estimated.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EmbeddedCover

```rust
pub struct EmbeddedCover {
    pub format: PictureFormat,
    pub len: u64,
    pub sha256: Sha256Digest,
    pub span: Option<ByteSpan>,
}
```

Where the embedded cover is and what it is, without its bytes.

The host fetches [`Self::span`](#embeddedcover) itself and hands it to the cover worker,
so a page of tag replies never carries pictures.

**Fields**

- `format`: The picture format, sniffed from its first bytes (not the declared
  MIME type).
- `len`: Picture size in bytes.
- `sha256`: SHA-256 of the picture bytes.
- `span`: Where the picture bytes lie in the file, when they are stored as is
  (not inside an unsynchronised ID3 tag).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RegionDigest

```rust
pub struct RegionDigest {
    pub version: u16,
    pub sha256: Sha256Digest,
    pub len: u64,
}
```

Digest of a file's metadata region.

The region is the bytes that hold tags and chapters and nothing else: an
MP4's `udta` and `meta` boxes, its chapter track tables and the chapter
title samples; an MP3's ID3v2 and ID3v1 tags; a FLAC's Vorbis comment,
picture and cue sheet blocks. Re-encoding the audio or moving it leaves
the digest alone unless the tags move with it; editing a tag changes it.

**Fields**

- `version`: How the digest was computed; [`REGION_HASH_VERSION`](#region_hash_version) today. A later
  version means stored digests are recomputed, not compared.
- `sha256`: SHA-256 over each region span in file order, each as its length (a
  big-endian `u64`) followed by its bytes.
- `len`: Total bytes the digest covers.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Sha256Digest

```rust
pub struct Sha256Digest(pub [u8; 32]);
```

A SHA-256 digest, written as 64 lowercase hex digits.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### TagEntry

```rust
pub struct TagEntry {
    pub field: TagField,
    pub value: String,
}
```

One text value from the file.

**Fields**

- `field`: Which field the value belongs to.
- `value`: The value; at most [`MAX_VALUE_BYTES`](#max_value_bytes) bytes, never containing NUL.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TagRecord

```rust
pub struct TagRecord {
    pub container: Container,
    pub entries: Vec<TagEntry>,
    pub chapters: Vec<Chapter>,
    pub chapter_source: Option<ChapterSource>,
    pub duration: Option<Duration>,
    pub codec: Option<AudioCodec>,
    pub format: AudioFormat,
    pub cover: Option<EmbeddedCover>,
    pub region: RegionDigest,
    pub truncated: bool,
}
```

The metadata one file carries in its own container.

Values are reported as the file holds them, without normalisation: the
resolver decides what they mean. Entries keep the file's order, and a
field may appear more than once (several artists, several genres).

**Fields**

- `container`: The container the file was read as.
- `entries`: Text values, in file order; at most [`MAX_ENTRIES`](#max_entries).
- `chapters`: Chapters, by start time; at most [`MAX_CHAPTERS`](#max_chapters).
- `chapter_source`: Where [`Self::chapters`](#tagrecord) came from; `None` exactly when there are none.
- `duration`: The running time, when the container states or implies one.
- `codec`: How the audio is encoded, when the container names an encoding the
  reader knows; `None` for an MP4 file with no audio track or with one
  in another encoding.
- `format`: Channels, sample rate and AAC profile of the audio, as far as the
  stream's headers state them.
- `cover`: The embedded cover picture, when there is one.
- `region`: Digest of the bytes that hold the metadata.
- `truncated`: True when entries, chapters or values were cut to fit the limits, so
  the record is not everything the file holds.

#### TagRecord::validate

```rust
pub fn validate(&self, file_size: u64) -> Result<(), RecordError>
```

Checks a record that came from the worker against the reader's
limits, so a compromised worker cannot hand the host more than an
honest reader would.

##### Arguments

* `file_size` - Size of the file the record describes, in bytes.

##### Errors

The first [`RecordError`](#recorderror) found.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ByteSpan

```rust
pub struct ByteSpan {
    pub offset: u64,
    pub len: u64,
}
```

A range of bytes in a file.

**Fields**

- `offset`: Offset of the first byte.
- `len`: Number of bytes.

#### ByteSpan::end

```rust
pub fn end(&self) -> Option<u64>
```

The offset one past the last byte, or `None` when that overflows.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

## Enums

### Poll

```rust
pub enum Poll {
    Need(Vec<ByteSpan>),
    Done(Result<TagRecord, TagError>),
}
```

What [`TagReader::poll`](#tagreaderpoll) wants next.

**Variants**

- `Need`: Supply each of these spans (in any order) with
  [`TagReader::supply`](#tagreadersupply), then poll again. Every span lies inside the
  file; there are between one and [`MAX_SPANS_PER_ROUND`](#max_spans_per_round).
- `Done`: The reader is done; later polls return the same answer.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AacProfile

```rust
pub enum AacProfile {
    Lc,
    He,
    HeV2,
    Xhe,
}
```

The AAC profiles a library tells apart, from the audio object type of
an MPEG-4 `AudioSpecificConfig`.

**Variants**

- `Lc`: AAC Low Complexity (object type 2), what most stores sell.
- `He`: HE-AAC: AAC with spectral band replication (object type 5).
- `HeV2`: HE-AAC v2: HE-AAC with parametric stereo (object type 29).
- `Xhe`: Extended HE-AAC, the USAC codec (object type 42).

#### AacProfile::as_str

```rust
pub fn as_str(self) -> &'static str
```

The profile's name as records and the web API spell it (`lc`, `he`,
`he_v2`, `xhe`), the same as its serialized form.

#### AacProfile::parse

```rust
pub fn parse(name: &str) -> Option<Self>
```

The profile [`Self::as_str`](#aacprofileas_str) spells `name`, or `None` for a name
this build does not know.

#### AacProfile::from_object_type

```rust
pub fn from_object_type(object_type: u8) -> Option<Self>
```

The profile of MPEG-4 audio object type `object_type`, when it is
one of the four.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### AudioCodec

```rust
pub enum AudioCodec {
    Aac,
    Alac,
    Flac,
    Mp3,
    Opus,
    Vorbis,
    Eac3,
    Ac4,
}
```

The audio encodings the reader tells apart.

The encoding decides whether a file is a lossless copy of its source,
which is what lets a library keep the better of two copies of one
edition (library data model ADR, section 3).

**Variants**

- `Aac`: MPEG-4 AAC (`mp4a` sample entry), lossy.
- `Alac`: Apple Lossless (`alac` sample entry).
- `Flac`: FLAC, in its own container or in MP4 (`fLaC` sample entry).
- `Mp3`: MPEG audio layer I, II or III, lossy.
- `Opus`: Opus, in Ogg or in MP4 (`Opus` sample entry), lossy.
- `Vorbis`: Vorbis, in Ogg, lossy.
- `Eac3`: Dolby Digital Plus (`ec-3` sample entry), lossy.
- `Ac4`: Dolby AC-4 (`ac-4` sample entry), lossy.

#### AudioCodec::as_str

```rust
pub fn as_str(self) -> &'static str
```

The codec's name as records and the web API spell it (`aac`,
`opus`, `eac3`, ...), the same as its serialized form.

#### AudioCodec::parse

```rust
pub fn parse(name: &str) -> Option<Self>
```

The codec [`Self::as_str`](#audiocodecas_str) spells `name`, or `None` for a name this
build does not know.

#### AudioCodec::is_lossless

```rust
pub fn is_lossless(self) -> bool
```

True when decoding gives back the source audio bit for bit.

#### AudioCodec::from_sample_entry

```rust
pub fn from_sample_entry(format: [u8; 4]) -> Option<Self>
```

The codec an MP4 audio sample entry of type `format` holds, when the
reader knows it.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### ChapterSource

```rust
pub enum ChapterSource {
    Mp4Track,
    Mp4Nero,
    Id3Chap,
    VorbisComment,
}
```

Which structure a file's chapters were read from.

**Variants**

- `Mp4Track`: A QuickTime chapter text track (what Apple players and Bookclerk use).
- `Mp4Nero`: A Nero `chpl` box, read when there is no chapter track.
- `Id3Chap`: ID3v2 `CHAP` frames.
- `VorbisComment`: Vorbis `CHAPTERnnn` / `CHAPTERnnnNAME` comments.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### Container

```rust
pub enum Container {
    Mp4,
    Mp3,
    Flac,
    Ogg,
}
```

The container formats the reader understands.

**Variants**

- `Mp4`: ISO base media (`.m4b`, `.m4a`, `.mp4`): `ilst` items, Nero `chpl`
  and QuickTime chapter tracks.
- `Mp3`: MPEG audio (`.mp3`): an ID3v2 tag at the start, or failing that an
  ID3v1 tag at the end.
- `Flac`: FLAC: Vorbis comments, pictures and `STREAMINFO`.
- `Ogg`: Ogg (`.opus`, `.ogg`, `.oga`) holding Opus or Vorbis: the stream's
  comment header and the last page's granule position.

#### Container::content_type

```rust
pub fn content_type(self) -> &'static str
```

The media type files of this container are stored and served as.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### DurationPrecision

```rust
pub enum DurationPrecision {
    Exact,
    Estimated,
}
```

How a [`Duration`](#duration) was found.

**Variants**

- `Exact`: Stated by the container (MP4 `mvhd`, FLAC `STREAMINFO`, an MP3
  Xing/Info/VBRI frame count), or every MP3 frame was counted.
- `Estimated`: Worked out from the MP3 frames in the first 64 KiB and the audio
  size: right for constant bit rate files, close for variable ones.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### PictureFormat

```rust
pub enum PictureFormat {
    Jpeg,
    Png,
    Webp,
    Gif,
    Bmp,
    Unknown,
}
```

The format of an embedded picture, from its signature.

**Variants**

- `Jpeg`: JPEG (`FF D8 FF`).
- `Png`: PNG.
- `Webp`: WebP (`RIFF....WEBP`).
- `Gif`: GIF.
- `Bmp`: BMP (`BM`).
- `Unknown`: None of the above.

#### PictureFormat::sniff

```rust
pub fn sniff(bytes: &[u8]) -> Self
```

The format whose signature `bytes` starts with.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### RecordError

```rust
pub enum RecordError {
    TooMany(&'static str),
    BadText(&'static str),
    ChapterOrder,
    ChapterSource,
    Duration,
    Codec,
    Format,
    CoverSpan,
    RegionLen,
    RegionVersion(u16),
}
```

Why a [`TagRecord`](#tagrecord) from the worker was refused.

**Variants**

- `TooMany`: More entries or chapters than the limits allow.
- `BadText`: A key, value or title is empty where it may not be, too long, or
  contains NUL.
- `ChapterOrder`: Chapters are not in start order.
- `ChapterSource`: `chapter_source` does not agree with whether there are chapters.
- `Duration`: A duration over [`MAX_DURATION_MS`](#max_duration_ms).
- `Codec`: A codec the container cannot hold (an MP3 or FLAC file always holds
  its own).
- `Format`: Channels or a sample rate of zero or over the limits, or an AAC
  profile on audio that is not AAC.
- `CoverSpan`: A cover span outside the file or of the wrong length.
- `RegionLen`: A region digest that covers more bytes than the file has.
- `RegionVersion`: A region digest of another version.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### TagField

```rust
pub enum TagField {
    Title,
    Subtitle,
    Album,
    Artist,
    AlbumArtist,
    Composer,
    Genre,
    Date,
    Comment,
    Description,
    LongDescription,
    Publisher,
    Copyright,
    Language,
    Track,
    Disc,
    Grouping,
    Freeform { namespace: Option<String>, name: String },
    Native { id: String },
}
```

The field a [`TagEntry`](#tagentry) fills.

The named variants are the fields every supported container has a
standard slot for. Everything else keeps the container's own key, so no
value is dropped for want of a name.

**Variants**

- `Title`: Track or book title (`©nam`, `TIT2`, `TITLE`).
- `Subtitle`: Subtitle (`TIT3`, `SUBTITLE`).
- `Album`: Album, which audiobook tools use for the book title (`©alb`, `TALB`, `ALBUM`).
- `Artist`: Artist, usually the author (`©ART`, `TPE1`, `ARTIST`).
- `AlbumArtist`: Album artist (`aART`, `TPE2`, `ALBUMARTIST`).
- `Composer`: Composer, which audiobook tools use for the narrator (`©wrt`, `TCOM`, `COMPOSER`).
- `Genre`: Genre (`©gen`, `TCON`, `GENRE`).
- `Date`: Release date or year, as written (`©day`, `TDRC`/`TYER`, `DATE`).
- `Comment`: Comment (`©cmt`, `COMM`, `COMMENT`).
- `Description`: Short description (`desc`, `DESCRIPTION`).
- `LongDescription`: Long description (`ldes`).
- `Publisher`: Publisher (`©pub`, `TPUB`, `PUBLISHER`, `ORGANIZATION`).
- `Copyright`: Copyright (`cprt`, `TCOP`, `COPYRIGHT`).
- `Language`: Language (`TLAN`, `LANGUAGE`).
- `Track`: Track number, as `n` or `n/total` (`trkn`, `TRCK`, `TRACKNUMBER`).
- `Disc`: Disc number, as `n` or `n/total` (`disk`, `TPOS`, `DISCNUMBER`).
- `Grouping`: Grouping (`©grp`, `GRP1`/`TIT1`, `GROUPING`).
- `Freeform`: A user-defined value: an MP4 `----` item, an ID3 `TXXX` frame, or a
  Vorbis comment key with no named field.
  - `namespace`: The MP4 `mean` (such as `com.apple.iTunes` or
    `com.pilabor.tone`); `None` for ID3 and Vorbis.
  - `name`: The item name, `TXXX` description or comment key.
- `Native`: A standard container field with no named variant, by its native key
  (an MP4 atom such as `©too`, an ID3 frame id such as `TENC`).
  - `id`: The atom or frame id.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### TagError

```rust
pub enum TagError {
    Unsupported,
    Malformed { container: &'static str, detail: &'static str },
    TooLarge,
    Unrequested,
    Id3(String),
}
```

Why a file's tags could not be read.

**Variants**

- `Unsupported`: The file is not a container the reader understands.
- `Malformed`: The container is malformed: a structure runs past its parent or the
  file, or a required box is missing.
  - `container`: The container being read.
  - `detail`: What was wrong.
- `TooLarge`: The metadata is larger than [`MAX_FETCH_BYTES`](#max_fetch_bytes) or needs more than
  [`MAX_ROUNDS`](#max_rounds) rounds.
- `Unrequested`: The caller supplied bytes the reader did not ask for, or the wrong
  number of them.
- `Id3`: The ID3v2 tag could not be parsed.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### read_all

```rust
pub fn read_all(file: &[u8]) -> Result<TagRecord, TagError>
```

Reads a whole in-memory file in one pass.

The reference the fuzz oracle compares [`TagReader`](#tagreader) against: the same
parser with every byte available up front.

#### Errors

The [`TagError`](#tagerror) the reader would report.

## Constants

### MAX_CHANNELS

```rust
pub const MAX_CHANNELS: u8 = 32;
```

Most channels a record may claim: above any layout an audiobook or a
surround mix uses, so a larger value means a corrupt header.

### MAX_DURATION_MS

```rust
pub const MAX_DURATION_MS: u64 = _;
```

Longest running time a record may claim: 10,000 hours, in milliseconds.

Far beyond any real audiobook; a larger value means a corrupt header.

### MAX_SAMPLE_RATE_HZ

```rust
pub const MAX_SAMPLE_RATE_HZ: u32 = 768_000;
```

Highest sample rate a record may claim, in hertz (768 kHz, the top of
any PCM or FLAC stream).

### REGION_HASH_VERSION

```rust
pub const REGION_HASH_VERSION: u16 = 1;
```

Version of the region digest this reader computes ([`RegionDigest::version`](#regiondigest)).

### MAX_ENTRIES

```rust
pub const MAX_ENTRIES: usize = 512;
```

Most text entries a record carries; the rest are dropped and the record
marked truncated.

### MAX_VALUE_BYTES

```rust
pub const MAX_VALUE_BYTES: usize = _;
```

Longest value an entry carries, in bytes; longer values are cut at a
character boundary and the record marked truncated.

### MAX_KEY_BYTES

```rust
pub const MAX_KEY_BYTES: usize = 256;
```

Longest key (freeform name, namespace or native id), in bytes; an entry
with a longer key is dropped.

### MAX_CHAPTERS

```rust
pub const MAX_CHAPTERS: usize = 4096;
```

Most chapters a record carries.

### MAX_CHAPTER_TITLE_BYTES

```rust
pub const MAX_CHAPTER_TITLE_BYTES: usize = 1024;
```

Longest chapter title, in bytes.

### MAX_FETCH_BYTES

```rust
pub const MAX_FETCH_BYTES: u64 = _;
```

Most bytes the reader asks for over one file. Real metadata, cover
included, is a few megabytes at most.

### MAX_ROUNDS

```rust
pub const MAX_ROUNDS: u32 = 128;
```

Most rounds of [`Poll::Need`](#poll) for one file.

### MAX_SPANS_PER_ROUND

```rust
pub const MAX_SPANS_PER_ROUND: usize = 64;
```

Most spans one [`Poll::Need`](#poll) asks for.

### MIN_FETCH_BYTES

```rust
pub const MIN_FETCH_BYTES: u64 = _;
```

Least the reader asks for at once when it only needs a header, so a walk
over small boxes takes few rounds.
