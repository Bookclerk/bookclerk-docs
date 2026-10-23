**bookclerk_config > pipeline_opts**

# Module: pipeline_opts

## Contents

**Enums**

- [`ChapterJsonMode`](#chapterjsonmode) - Whether / which chapter JSON sidecars to write beside acquired audio.
- [`OutputFormat`](#outputformat) - Post-download output formatting. Default is enriched M4B (tags + chapters).

---

## bookclerk_config::pipeline_opts::ChapterJsonMode

*Enum*

Whether / which chapter JSON sidecars to write beside acquired audio.

Default is `Self::Off`. Replaces the old `save_chapter_json` bool +
`chapter_layout` pair for *sidecar output* (Audible API fetch layout is
chosen separately when chapters are needed for embedding).

**Variants:**
- `Off` - Do not write chapter JSON sidecars.
- `Flat` - Write `chapters.flat.json` only.
- `Tree` - Write `chapters.tree.json` (nested) only.
- `Both` - Write both flat and tree sidecars.

**Methods:**

- `fn wants_flat(self: Self) -> bool` - True when a flat `chapters.flat.json` sidecar should be written.
- `fn wants_tree(self: Self) -> bool` - True when a nested `chapters.tree.json` sidecar should be written.
- `fn wants_any(self: Self) -> bool` - True when any chapter JSON sidecar mode other than `Self::Off` is set.
- `fn parse(raw: &str) -> Option<Self>` - Parse `off|flat|tree|both`, or legacy bool-ish values (`true`→tree, `false`→off).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ChapterJsonMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ChapterJsonMode`
- **Default**
  - `fn default() -> ChapterJsonMode`



## bookclerk_config::pipeline_opts::OutputFormat

*Enum*

Post-download output formatting. Default is enriched M4B (tags + chapters).

**Variants:**
- `EnrichedM4b` - Remux/transcode to M4B and embed metadata/chapters (default).
- `SingleMp3` - Single MP3 file (LAME re-encode).
- `SplitMp3ByChapter` - One MP3 per chapter (after M4B chapter split).
- `SplitMp3BySize` - Split MP3 by target file size (see `split_mp3_max_mb`).
- `Opus` - Opus output (not yet implemented).
- `None` - Leave store-delivered media as-is (no remux/transcode/re-encode).

**Methods:**

- `fn parse(raw: &str) -> Option<Self>` - Parse a TOML / CLI format id (`enriched_m4b`, `mp3`, `none`, …).
- `fn wants_mp3(self: Self) -> bool` - True when the format re-encodes to MP3 (single or split).
- `fn wants_split_by_chapter(self: Self) -> bool` - True when output is one MP3 file per chapter.
- `fn wants_split_by_size(self: Self) -> bool` - True when output is split into MP3 parts by target size.
- `fn is_noop(self: Self) -> bool` - True when store-delivered media is left as-is (no remux/transcode).
- `fn wants_opus(self: Self) -> bool` - True when Opus packaging is selected (not yet implemented).
- `fn prefers_m4b_container(self: Self) -> bool` - True when the pipeline prefers an M4B intermediate (enriched or chapter-split).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputFormat) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> OutputFormat`
- **Default**
  - `fn default() -> OutputFormat`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



