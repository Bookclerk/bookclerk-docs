# bookclerk_library::quality

When a store's offer is a better file than the one the library holds.

The quality-upgrade check asks a store which stream an account would
download for a title today (`ContentSource.qualityOffers`) and compares
it with the original copy the library keeps, from the stream details the
acquire probe stored (`edition_files`). [`is_upgrade`](#is_upgrade) is the rule; the
daemon's upgrade scan decides what to ask and queues the downloads.

| Item | Kind | Summary |
| --- | --- | --- |
| [`StreamQuality`](#streamquality) | struct | A stream as the comparison sees it: the codec, the AAC profile when the codec is AAC, and the average bitrate. |
| [`is_upgrade`](#is_upgrade) | function | True when downloading `offer` would give a better file than `held`. |
| [`held_stream`](#held_stream) | function | The stream the library holds for an edition, from its audio files: the original-quality copy's files (what the store delivered, before any re-encode), all in one codec, at the lowest bitrate among them. |
| [`SAME_CLASS_STEP_PERCENT`](#same_class_step_percent) | constant | The bitrate step that makes the same class of codec worth downloading again: an offer must be at least this many hundredths of the held bitrate (115 = 1.15×). |

## Structs

### StreamQuality

```rust
pub struct StreamQuality {
    pub codec: AudioCodec,
    pub aac_profile: Option<AacProfile>,
    pub bitrate_kbps: Option<u32>,
}
```

A stream as the comparison sees it: the codec, the AAC profile when the
codec is AAC, and the average bitrate.

**Fields**

- `codec`: The audio encoding.
- `aac_profile`: The AAC profile; `None` for other codecs or when not known.
- `bitrate_kbps`: Average bitrate in kbit/s; `None` when not known.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### is_upgrade

```rust
pub fn is_upgrade(held: &StreamQuality, offer: &StreamQuality) -> bool
```

True when downloading `offer` would give a better file than `held`.

- A lossless offer beats any lossy file, whatever the bitrates; nothing
  beats a lossless file.
- A better class of codec (MP3, then AAC-LC / HE-AAC / Vorbis, then
  xHE-AAC / Opus) wins at the same or a higher bitrate.
- The same class wins at [`SAME_CLASS_STEP_PERCENT`](#same_class_step_percent) of the held bitrate
  or more.
- Anything else, including an offer or a held file whose bitrate is not
  known, or a Dolby codec on either side, is not an upgrade.

### held_stream

```rust
pub fn held_stream(files: &[ProbedAudioFile]) -> Option<StreamQuality>
```

The stream the library holds for an edition, from its audio files: the
original-quality copy's files (what the store delivered, before any
re-encode), all in one codec, at the lowest bitrate among them.

#### Returns

`None` when no original-quality file was probed with a codec, or when
its files disagree on codec or AAC profile.

## Constants

### SAME_CLASS_STEP_PERCENT

```rust
pub const SAME_CLASS_STEP_PERCENT: u64 = 115;
```

The bitrate step that makes the same class of codec worth downloading
again: an offer must be at least this many hundredths of the held
bitrate (115 = 1.15×).
