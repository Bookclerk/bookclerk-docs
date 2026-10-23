**bookclerk_source > options**

# Module: options

## Contents

**Structs**

- [`DownloadOptions`](#downloadoptions) - Options for acquire / fetch packaging and naming.

---

## bookclerk_source::options::DownloadOptions

*Struct*

Options for acquire / fetch packaging and naming.

Store-specific ingest knobs live on each [`crate::ContentSource`](traits.md#bookclerk_sourcetraitscontentsource) instance
(parsed from `[sources.<id>]` at registration). `quality` remains here so
Audible’s download helpers can overlay the plugin bitrate for a single fetch.

**Fields:**
- `quality: bookclerk_config::AudioQuality` - Optional per-fetch audio quality overlay (set by plugins that need it).
- `format: bookclerk_config::OutputFormat` - Post-download packaging format (M4B / MP3 / passthrough).
- `widevine: bool` - Prefer Widevine/CENC download when the store offers it.
- `xhe_aac: bool` - Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm: Option<std::path::PathBuf>` - Optional local Widevine `.wvd` path (absolute or under files dir).
- `widevine_cdm_provider: Option<String>` - Remote L3 CDM provider URL (`None` = classic Libation AudibleCdm; empty/`off` = disable).
- `naming_profile: bookclerk_config::NamingProfile` - Path-template profile; per-field template overrides win when set.
- `folder_template: Option<String>` - Optional folder-path template override; `None` uses the naming profile.
- `file_template: Option<String>` - Optional file-stem template override; `None` uses the naming profile.
- `download_cover: bool` - When true, download a cover JPEG alongside audio.
- `download_pdf: bool` - When true, download a companion PDF when the store exposes one.
- `create_cue: bool` - When true, write a `.cue` sidecar from API chapters.
- `fixup_metadata: bool` - When true, embed tags, cover, and chapters after packaging.
- `chapter_json: bookclerk_config::ChapterJsonMode` - Which chapter JSON sidecars to write (`off` / `flat` / `tree` / `both`).
- `save_metadata_json: bool` - When true, persist raw catalog API JSON as `metadata.json`.
- `cover_size: String` - Cover image size request (`500`, `1215`, or `native`).
- `chapter_layout: String` - Preferred Audible chapter API layout when fetching (`tree` or `flat`).
- `overwrite_existing: bool` - When true, re-acquire even if media already exists at the destination.
- `split_files_by_chapter: bool` - When true, also split packaged output into per-chapter files.
- `split_mp3_max_mb: u32` - Max MP3 part size in MiB when format is `split_mp3_by_size`.
- `chapter_file_template: Option<String>` - Optional per-chapter file-stem template.
- `chapter_title_template: Option<String>` - Optional embedded chapter-title template.
- `minimum_file_duration_minutes: u32` - Drop chapter splits shorter than this many minutes (`0` = keep all).
- `combine_nested_chapter_titles: bool` - When true, flatten nested chapter titles into a single path segment.
- `merge_opening_and_end_credits: bool` - When true, merge opening/end credit chapters into adjacent chapters.
- `strip_unabridged: bool` - When true, strip an "Unabridged" suffix from titles used in naming.
- `strip_audible_brand_audio: bool` - When true, trim Audible brand intro/outro from the remux window.
- `download_clips_bookmarks: bool` - When true, download Audible clips/bookmarks sidecars when offered.
- `retain_aax_file: bool` - When true, keep the encrypted download in storage (`RetainAaxFile`).
- `download_speed_limit_kbps: u32` - Fetch speed cap in KB/s (`0` = unlimited).
- `lame: bookclerk_config::LameConfig` - LAME encoder knobs used when packaging to MP3.
- `max_sample_rate: Option<u32>` - Optional ceiling for output sample rate in Hz (`None` = leave source rate).
- `creation_time: bookclerk_config::FileTimestampMode` - How to set the file creation / birth timestamp after acquire.
- `last_write_time: bookclerk_config::FileTimestampMode` - How to set the file last-write / mtime after acquire.
- `replacement_characters: Vec<bookclerk_config::ReplacementRule>` - Resolved find/replace rules for path-segment sanitisation.
- `path_limits: bookclerk_config::PathLimits` - Filesystem / object-store path length limits for storage keys.
- `save_podcasts_to_parent_folder: bool` - Save podcast episodes under the parent show folder.

**Methods:**

- `fn for_destination(cfg: &Config, kind: OutputBackendKind, naming: &DestinationNaming, prefix: &str) -> Self` - Build acquire options stamped for one output destination.
- `fn effective_output(self: &Self) -> OutputFormat` - Resolved post-processing format.
- `fn wants_mp3(self: &Self) -> bool` - True when the effective format re-encodes to MP3.
- `fn wants_split_by_chapter(self: &Self) -> bool` - True when output should be split into per-chapter files.
- `fn is_noop_output(self: &Self) -> bool` - True when store-delivered media is left as-is (no remux/transcode).
- `fn wants_opus(self: &Self) -> bool` - True when Opus packaging is selected (not yet implemented).
- `fn wants_split_by_size(self: &Self) -> bool` - True when output is split into MP3 parts by target size.
- `fn chapter_json_flat(self: &Self) -> bool` - True when a flat `chapters.flat.json` sidecar should be written.
- `fn chapter_json_tree(self: &Self) -> bool` - True when a nested `chapters.tree.json` sidecar should be written.
- `fn wants_chapter_json(self: &Self) -> bool` - True when any chapter JSON sidecar should be written.
- `fn naming_templates(self: &Self) -> ResolvedNamingTemplates` - Resolve folder / file / chapter-file templates from the naming profile
- `fn from_fetch_options(fetch: &abi::FetchOptions) -> Self` - Guest-side reconstruction: host defaults overlaid with the fetch knobs
- `fn apply_fetch_options(self: & mut Self, fetch: &abi::FetchOptions)` - Overlays wire fetch knobs onto these options (packaging knobs untouched).

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &DownloadOptions) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DownloadOptions`
- **Default**
  - `fn default() -> Self`
- **From**
  - `fn from(cfg: &OutputConfig) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(cfg: &Config) -> Self` - Global `[output]` naming with no destination prefix: what a listing,
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



