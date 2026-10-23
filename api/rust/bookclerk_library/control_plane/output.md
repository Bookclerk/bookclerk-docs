**bookclerk_library > control_plane > output**

# Module: control_plane::output

## Contents

**Structs**

- [`OutputSettingsV1`](#outputsettingsv1) - Typed `core.output` document, schema version 1.

**Enums**

- [`OutputReplace`](#outputreplace) - Outcome of [`replace_output`](output.md#bookclerk_librarycontrol_planeoutputreplace_output).

**Functions**

- [`import_output_if_absent`](#import_output_if_absent) - Inserts `[output]` when the cluster document is absent.
- [`is_output_setting_key`](#is_output_setting_key) - True when `key` is a `core.output` setting or alias.
- [`load_output`](#load_output) - Loads the cluster output document.
- [`overlay_output`](#overlay_output) - Copies the authoritative output document onto `config` for `cluster_id`.
- [`portable_output_seed`](#portable_output_seed) - Copies `[output]` with a `widevine_cdm` under `files_dir` made relative again.
- [`replace_output`](#replace_output) - Replaces the cluster output document.

**Constants**

- [`MAX_OUTPUT_REPLACEMENT_RULES`](#max_output_replacement_rules) - Maximum number of explicit path replacement rules.
- [`MAX_OUTPUT_TEMPLATE_BYTES`](#max_output_template_bytes) - Maximum byte length of a naming template.
- [`OUTPUT_NAMESPACE`](#output_namespace) - Namespace for the migrated shared `[output]` domain.
- [`OUTPUT_SCHEMA_VERSION`](#output_schema_version) - Schema version of [`OutputSettingsV1`](output.md#bookclerk_librarycontrol_planeoutputoutputsettingsv1).
- [`OUTPUT_SETTING_ALIASES`](#output_setting_aliases) - Write-only aliases `OutputSettingsV1::set` also accepts.
- [`OUTPUT_SETTING_KEYS`](#output_setting_keys) - Every `output.*` key `OutputSettingsV1::get` and `OutputSettingsV1::set`

---

## bookclerk_library::control_plane::output::MAX_OUTPUT_REPLACEMENT_RULES

*Constant*: `usize`

Maximum number of explicit path replacement rules.



## bookclerk_library::control_plane::output::MAX_OUTPUT_TEMPLATE_BYTES

*Constant*: `usize`

Maximum byte length of a naming template.



## bookclerk_library::control_plane::output::OUTPUT_NAMESPACE

*Constant*: `&str`

Namespace for the migrated shared `[output]` domain.



## bookclerk_library::control_plane::output::OUTPUT_SCHEMA_VERSION

*Constant*: `i64`

Schema version of [`OutputSettingsV1`](output.md#bookclerk_librarycontrol_planeoutputoutputsettingsv1).



## bookclerk_library::control_plane::output::OUTPUT_SETTING_ALIASES

*Constant*: `&[&str]`

Write-only aliases `OutputSettingsV1::set` also accepts.

`output.save_chapter_json` takes a boolean or a chapter-JSON mode;
`output.split_files_by_chapter = true` selects the per-chapter MP3 format.



## bookclerk_library::control_plane::output::OUTPUT_SETTING_KEYS

*Constant*: `&[&str]`

Every `output.*` key `OutputSettingsV1::get` and `OutputSettingsV1::set`
accept, in display order.



## bookclerk_library::control_plane::output::OutputReplace

*Enum*

Outcome of [`replace_output`](output.md#bookclerk_librarycontrol_planeoutputreplace_output).

**Variants:**
- `Applied(Box<super::ConfigurationDocument<OutputSettingsV1>>)` - The new document is committed.
- `Conflict{ current_revision: i64 }` - `expected_revision` was not current. The stored document is unchanged.
- `Replayed{ revision: i64 }` - This operation id already committed at `revision`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OutputReplace`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputReplace) -> bool`



## bookclerk_library::control_plane::output::OutputSettingsV1

*Struct*

Typed `core.output` document, schema version 1.

Field meanings match [`OutputConfig`](../../bookclerk_config/output.md#bookclerk_configoutputoutputconfig). Unknown fields are rejected, inside
the nested `lame` and `replacement_characters` objects too, so a document
written by a newer binary under the same schema version fails closed
instead of being silently truncated on the next write.

**Fields:**
- `format: bookclerk_config::OutputFormat` - Post-acquire packaging format.
- `widevine: bool` - Prefer Widevine/CENC.
- `xhe_aac: bool` - Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm: Option<std::path::PathBuf>` - Optional `.wvd` path; relative paths resolve under each host's files dir.
- `widevine_cdm_provider: Option<String>` - Remote L3 CDM provider (`None` = AudibleCdm, empty/`off` disables).
- `naming_profile: bookclerk_config::NamingProfile` - Named path-template preset.
- `folder_template: Option<String>` - Folder-path template override.
- `file_template: Option<String>` - File-stem template override.
- `chapter_file_template: Option<String>` - Per-chapter file-stem template override.
- `chapter_title_template: Option<String>` - Embedded chapter-title template.
- `download_cover: bool` - Save cover JPEG alongside audio.
- `download_pdf: bool` - Download companion PDF when available.
- `create_cue: bool` - Write a `.cue` sidecar.
- `fixup_metadata: bool` - Embed tags, cover, and chapters natively.
- `chapter_json: bookclerk_config::ChapterJsonMode` - Chapter JSON sidecars.
- `save_metadata_json: bool` - Persist raw catalog API JSON.
- `cover_size: String` - Cover image size (`500`, `1215`, another pixel size, or `native`).
- `chapter_layout: String` - Preferred chapter API layout (`tree`, `flat`, or `both`).
- `overwrite_existing: bool` - Re-acquire when media already exists at the destination.
- `multi_destination: bookclerk_config::MultiDestinationMode` - Policy when only some enabled destinations have a title.
- `bad_book_action: bookclerk_config::BadBookAction` - Operator action when a title fails to acquire.
- `split_mp3_max_mb: u32` - Max MP3 part size in MiB for `split_mp3_by_size`.
- `minimum_file_duration_minutes: u32` - Drop chapter splits shorter than this many minutes.
- `combine_nested_chapter_titles: bool` - Flatten nested chapter titles.
- `merge_opening_and_end_credits: bool` - Merge opening/end credit chapters into neighbours.
- `strip_unabridged: bool` - Strip an "Unabridged" suffix from titles used in naming.
- `strip_audible_brand_audio: bool` - Trim Audible brand intro/outro audio.
- `download_clips_bookmarks: bool` - Download Audible clips/bookmarks sidecars.
- `retain_aax_file: bool` - Keep the encrypted download.
- `download_speed_limit_kbps: u32` - Fetch speed cap in KB/s (`0` = unlimited).
- `lame: bookclerk_config::LameConfig` - LAME encoder knobs.
- `max_sample_rate: Option<u32>` - Optional output sample-rate ceiling in Hz.
- `creation_time: bookclerk_config::FileTimestampMode` - File creation timestamp policy.
- `last_write_time: bookclerk_config::FileTimestampMode` - File last-write timestamp policy.
- `path_sanitization: bookclerk_config::PathSanitizationMode` - Sanitization profile used when no explicit rules are set.
- `replacement_characters: Vec<bookclerk_config::ReplacementRule>` - Explicit find/replace rules for path segments.
- `max_filename_length: u32` - Max length per path segment (`0` disables truncation).

**Methods:**

- `fn from_config(config: &OutputConfig) -> Result<Self>` - Copies the transitional `[output]` table, rejecting values the domain forbids.
- `fn from_seed_lenient(config: &OutputConfig) -> (Self, Vec<&'static str>)` - Builds the document from a seed `[output]` table (TOML / environment)
- `fn apply_to(self: &Self, config: & mut OutputConfig, files_dir: Option<&Path>)` - Writes the document onto the in-memory `[output]` table.
- `fn validate(self: &Self) -> Result<()>` - Rejects values outside the domain bounds.
- `fn get(self: &Self, key: &str) -> Option<String>` - Formats one `output.*` key for display.
- `fn set(self: & mut Self, key: &str, value: &str) -> Result<()>` - Assigns one `output.*` key from its string form, then validates.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputSettingsV1) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OutputSettingsV1`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::output::import_output_if_absent

*Function*

Inserts `[output]` when the cluster document is absent.

An existing document is parsed and returned without reading the seed, so
an obsolete or out-of-range TOML/environment value cannot fail startup or
replace the stored document.

# Errors

Returns an error when the stored document is invalid, or when the document
is absent and the actor may not import. An absent document is seeded with
[`OutputSettingsV1::from_seed_lenient`]: an out-of-range seed field is
logged and replaced by its default rather than failing startup.

```rust
fn import_output_if_absent(store: &crate::store::LibraryStore, actor: &super::ConfigActor, seed: &bookclerk_config::OutputConfig, operation_id: &str) -> crate::error::Result<super::ConfigurationDocument<OutputSettingsV1>>
```



## bookclerk_library::control_plane::output::is_output_setting_key

*Function*

True when `key` is a `core.output` setting or alias.

```rust
fn is_output_setting_key(key: &str) -> bool
```



## bookclerk_library::control_plane::output::load_output

*Function*

Loads the cluster output document.

# Errors

Returns an error when the document is missing, unsupported, or invalid.

```rust
fn load_output(store: &crate::store::LibraryStore) -> crate::error::Result<super::ConfigurationDocument<OutputSettingsV1>>
```



## bookclerk_library::control_plane::output::overlay_output

*Function*

Copies the authoritative output document onto `config` for `cluster_id`.

Revisions are only comparable within one cluster. Callers that switch
databases pass the new cluster id explicitly.

```rust
fn overlay_output(config: & mut bookclerk_config::Config, output: &super::ConfigurationDocument<OutputSettingsV1>, cluster_id: &str)
```



## bookclerk_library::control_plane::output::portable_output_seed

*Function*

Copies `[output]` with a `widevine_cdm` under `files_dir` made relative again.

`Config::load` resolves a relative CDM path under the loading host's files
directory. The shared document keeps it relative, so every host rebases it
under its own `BOOKCLERK_FILES_DIR` in `OutputSettingsV1::apply_to`. A
path outside `files_dir` (or equal to it) is kept as written.

# Arguments

* `seed` - The loaded `[output]` table.
* `files_dir` - The loading host's files directory.

```rust
fn portable_output_seed(seed: &bookclerk_config::OutputConfig, files_dir: &std::path::Path) -> bookclerk_config::OutputConfig
```



## bookclerk_library::control_plane::output::replace_output

*Function*

Replaces the cluster output document.

# Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

```rust
fn replace_output(store: &crate::store::LibraryStore, actor: &super::ConfigActor, expected_revision: i64, body: &OutputSettingsV1, operation_id: &str) -> crate::error::Result<OutputReplace>
```



