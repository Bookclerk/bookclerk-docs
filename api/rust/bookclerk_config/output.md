**bookclerk_config > output**

# Module: output

## Contents

**Structs**

- [`DestinationNaming`](#destinationnaming) - Optional naming overrides for one destination plugin.
- [`OutputConfig`](#outputconfig) - Acquire packaging (`[output]`).

**Enums**

- [`BadBookAction`](#badbookaction) - How to handle acquire failures (`BadBook` setting).
- [`MultiDestinationMode`](#multidestinationmode) - Behavior when some output destinations already have a title and others do not.
- [`OutputBackendKind`](#outputbackendkind) - Which output destination plugin is active.

**Functions**

- [`normalize_storage_prefix`](#normalize_storage_prefix) - Normalize a storage key prefix: empty stays empty; otherwise ensure a trailing `/`.

---

## bookclerk_config::output::BadBookAction

*Enum*

How to handle acquire failures (`BadBook` setting).

**Variants:**
- `Ask` - Prompt the operator (interactive CLI / GUI) for how to proceed (default).
- `Abort` - Stop the acquire batch on the first hard failure.
- `Retry` - Retry the failed title before giving up.
- `Ignore` - Skip the failed title and continue the batch.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &BadBookAction) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> BadBookAction`
- **Default**
  - `fn default() -> BadBookAction`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::output::DestinationNaming

*Struct*

Optional naming overrides for one destination plugin.

**Fields:**
- `naming_profile: Option<crate::naming_profile::NamingProfile>` - Override `[output].naming_profile` for this destination when set.
- `folder_template: Option<String>` - Optional folder-path template override (Libation tag syntax).
- `file_template: Option<String>` - Optional file-stem template override (no extension).
- `chapter_file_template: Option<String>` - Optional per-chapter file-stem template when splitting by chapter.

**Methods:**

- `fn resolve_over(self: &Self, global: &OutputConfig) -> ResolvedNamingTemplates` - Merge destination overrides over global `[output]` naming.
- `fn effective_profile(self: &Self, global: &OutputConfig) -> NamingProfile` - Profile after destination override (else global).
- `fn effective_folder_template(self: &Self, global: &OutputConfig) -> Option<String>` - Destination folder template, falling back to global `[output]`.
- `fn effective_file_template(self: &Self, global: &OutputConfig) -> Option<String>` - Destination file-stem template, falling back to global `[output]`.
- `fn effective_chapter_file_template(self: &Self, global: &OutputConfig) -> Option<String>` - Destination chapter-file template, falling back to global `[output]`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DestinationNaming`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &DestinationNaming) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> DestinationNaming`



## bookclerk_config::output::MultiDestinationMode

*Enum*

Behavior when some output destinations already have a title and others do not.

**Variants:**
- `SyncMissing` - Copy the existing object from a present destination into missing ones
- `RefetchMissing` - Re-run acquire fetch/encode, but write only to destinations that lack
- `RefetchAll` - Re-run acquire and write to every enabled destination.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &MultiDestinationMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> MultiDestinationMode`
- **Default**
  - `fn default() -> MultiDestinationMode`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::output::OutputBackendKind

*Enum*

Which output destination plugin is active.

**Variants:**
- `Local` - Filesystem destination (the `local` plugin; default primary).
- `S3` - Object-store destination (the `s3` plugin).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputBackendKind) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> OutputBackendKind`
- **Default**
  - `fn default() -> OutputBackendKind`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::output::OutputConfig

*Struct*

Acquire packaging (`[output]`).

Destination plugins (`local`, `s3`, …) declare their own `[[settings]]`
(root, bucket, prefix, per-destination naming overrides) and are
configured on their plugin instances; nothing destination-specific lives
in `config.toml`.

**Fields:**
- `format: crate::pipeline_opts::OutputFormat` - Post-acquire packaging format (default: enriched M4B).
- `widevine: bool` - Prefer Widevine/CENC (also enables Adrm→Widevine fallback; auto-provisions L3 CDM).
- `xhe_aac: bool` - Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm: Option<std::path::PathBuf>` - Optional local Widevine `.wvd` path (absolute or relative to `BOOKCLERK_FILES_DIR`).
- `widevine_cdm_provider: Option<String>` - Remote L3 CDM provider. `None` uses classic Libation AudibleCdm; empty/`off` disables.
- `naming_profile: crate::naming_profile::NamingProfile` - Named path-template preset (`audiobookshelf` default, or `classic`).
- `folder_template: Option<String>` - Optional folder-path template override; `None` uses the naming profile default.
- `file_template: Option<String>` - Optional file-stem template override; `None` uses the naming profile default.
- `download_cover: bool` - Save cover JPEG alongside audio.
- `download_pdf: bool` - Download companion PDF when available.
- `create_cue: bool` - Write a `.cue` sidecar from API chapters.
- `fixup_metadata: bool` - Embed tags, cover, and chapters natively.
- `chapter_json: crate::pipeline_opts::ChapterJsonMode` - Chapter JSON sidecars: `off` | `flat` | `tree` | `both`.
- `save_chapter_json: Option<bool>` - Deprecated: use `Self::chapter_json`.
- `save_metadata_json: bool` - Persist raw catalog API JSON (`metadata.json`).
- `cover_size: String` - Cover image size for download/embed (`500`, `1215`, or `native`).
- `chapter_layout: String` - Preferred Audible chapter API layout when fetching (`tree` or `flat`).
- `overwrite_existing: bool` - Re-acquire when media already exists at the destination.
- `multi_destination: MultiDestinationMode` - When multiple destinations are enabled and only some already have the
- `in_progress: Option<std::path::PathBuf>` - Scratch directory for in-progress work; relative to files_dir.
- `bad_book_action: BadBookAction` - Operator action when a title fails to acquire (`ask` / `abort` / `retry` / `ignore`).
- `split_mp3_max_mb: u32` - Max MP3 part size in MiB when format is `split_mp3_by_size`.
- `chapter_file_template: Option<String>` - Optional per-chapter file-stem template; `None` uses the naming profile default.
- `chapter_title_template: Option<String>` - Optional embedded chapter-title template when rewriting chapter names.
- `minimum_file_duration_minutes: u32` - Drop chapter splits shorter than this many minutes (`0` = keep all).
- `combine_nested_chapter_titles: bool` - When true, flatten nested chapter titles into a single path segment.
- `merge_opening_and_end_credits: bool` - When true, merge opening/end credit chapters into adjacent content chapters.
- `strip_unabridged: bool` - When true, strip an "Unabridged" suffix from titles used in naming.
- `strip_audible_brand_audio: bool` - When true, trim Audible brand intro/outro audio from the remux window.
- `download_clips_bookmarks: bool` - When true, download Audible clips/bookmarks sidecars when the store offers them.
- `retain_aax_file: bool` - Keep encrypted download in storage (`RetainAaxFile`).
- `download_speed_limit_kbps: u32` - Fetch speed cap in KB/s (`0` = unlimited).
- `lame: crate::extras::LameConfig` - LAME encoder knobs used when packaging to MP3.
- `max_sample_rate: Option<u32>` - Optional ceiling for output sample rate in Hz (`None` = leave source rate).
- `creation_time: crate::extras::FileTimestampMode` - How to set the file creation / birth timestamp after acquire.
- `last_write_time: crate::extras::FileTimestampMode` - How to set the file last-write / mtime after acquire.
- `path_sanitization: crate::extras::PathSanitizationMode` - Auto / Windows / POSIX / S3 sanitisation profile when replacement rules are empty.
- `replacement_characters: Vec<crate::extras::ReplacementRule>` - Explicit find/replace rules for path segments (wins over `Self::path_sanitization`).
- `max_filename_length: u32` - Max length per path segment. Default 255; `0` disables truncation.

**Methods:**

- `fn effective_format(self: &Self) -> OutputFormat` - Packaging format used for acquire (currently the raw `Self::format` field).
- `fn effective_chapter_json(self: &Self) -> ChapterJsonMode` - Resolved chapter JSON sidecar mode (default off).
- `fn resolve_naming_templates(self: &Self) -> ResolvedNamingTemplates` - Resolve folder / file / chapter-file templates.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OutputConfig`



## bookclerk_config::output::normalize_storage_prefix

*Function*

Normalize a storage key prefix: empty stays empty; otherwise ensure a trailing `/`.

```rust
fn normalize_storage_prefix(prefix: &str) -> String
```



