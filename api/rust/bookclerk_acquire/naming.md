**bookclerk_acquire > naming**

# Module: naming

## Contents

**Structs**

- [`NamingContext`](#namingcontext) - Metadata available to naming templates.

**Functions**

- [`audio_basename`](#audio_basename) - Basename of the acquired audio file (for `.cue` `FILE` lines).
- [`chapter_storage_key`](#chapter_storage_key) - Storage key for a split chapter file.
- [`chapter_storage_key_with_folder`](#chapter_storage_key_with_folder) - Chapter storage key with a separate folder naming context (podcast parent folder).
- [`default_storage_key`](#default_storage_key) - Build a relative storage key for a acquired title using the default
- [`resolve_templates`](#resolve_templates) - Resolve optional template overrides against a naming profile.
- [`sidecar_key`](#sidecar_key) - Replace the audio file extension on a storage key with a sidecar suffix.
- [`storage_key`](#storage_key) - Build a storage key from optional classic Libation folder/file templates.
- [`storage_key_with_contexts`](#storage_key_with_contexts) - Build a storage key using separate folder and file naming contexts.
- [`storage_key_with_rules`](#storage_key_with_rules) - Like [`storage_key`](naming.md#bookclerk_acquirenamingstorage_key) but applies classic `ReplacementCharacters`.
- [`swap_audio_extension`](#swap_audio_extension) - Replace the audio file extension on a storage key (e.g. `.m4b` → `.mp3`).

---

## bookclerk_acquire::naming::NamingContext

*Struct*

Metadata available to naming templates.

**Fields:**
- `asin: String` - Primary product id (Audible ASIN, ISBN, …) for this job.
- `title: String` - Display title used for naming and metadata.
- `subtitle: Option<String>` - Optional subtitle substituted into naming templates.
- `authors: Option<String>` - Author list for naming templates and tags.
- `narrators: Option<String>` - Narrator list for naming templates and tags.
- `series: Option<String>` - Series name for naming templates, when present.
- `series_index: Option<String>` - Series position string for naming templates, when present.
- `series_asin: Option<String>` - Podcast parent / series ASIN for `SavePodcastsToParentFolder`.
- `year_published: Option<i32>` - Publication year for `<year>` (from `published_at` when known).
- `account_id: Option<String>` - Store account id that owns this title.
- `account_nickname: Option<String>` - Operator-facing account label for `<account nickname>` templates.
- `locale: Option<String>` - Marketplace / locale code for naming templates.
- `language: Option<String>` - Content language code for naming templates.
- `publisher: Option<String>` - Publisher name for naming templates, when known.
- `categories: Option<String>` - Category / genre path list for naming templates.
- `length_minutes: Option<i64>` - Runtime in minutes for naming templates, when known.
- `bitrate: Option<i64>` - Audio bitrate label substituted into naming templates, when known.
- `samplerate: Option<i64>` - Audio sample rate in Hz for naming templates, when known.
- `channels: Option<i64>` - Channel count for naming templates, when known.
- `codec: Option<String>` - Audio codec label for naming templates, when known.
- `is_abridged: bool` - Whether the edition is abridged (template boolean).
- `content_kind: Option<String>` - Title kind (`book`, `episode`, …) for naming templates.
- `chapter_number: Option<u32>` - Chapter-specific: 1-based index.
- `chapter_count: Option<u32>` - Total chapter count when naming a split chapter file.
- `chapter_title: Option<String>` - Chapter title when naming a split chapter file.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NamingContext`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> NamingContext`



## bookclerk_acquire::naming::audio_basename

*Function*

Basename of the acquired audio file (for `.cue` `FILE` lines).

```rust
fn audio_basename(storage_key: &str) -> String
```



## bookclerk_acquire::naming::chapter_storage_key

*Function*

Storage key for a split chapter file.

```rust
fn chapter_storage_key(ctx: &NamingContext, folder_template: Option<&str>, chapter_file_template: Option<&str>, replacement_rules: &[bookclerk_config::ReplacementRule], chapter_number: usize, chapter_title: &str, ext: &str) -> String
```



## bookclerk_acquire::naming::chapter_storage_key_with_folder

*Function*

Chapter storage key with a separate folder naming context (podcast parent folder).

```rust
fn chapter_storage_key_with_folder(folder_ctx: &NamingContext, file_ctx: &NamingContext, folder_template: Option<&str>, chapter_file_template: Option<&str>, replacement_rules: &[bookclerk_config::ReplacementRule], chapter_number: usize, chapter_title: &str, ext: &str, path_limits: bookclerk_config::PathLimits) -> String
```



## bookclerk_acquire::naming::default_storage_key

*Function*

Build a relative storage key for a acquired title using the default
[`NamingProfile`](../bookclerk_config/naming_profile.md#bookclerk_confignaming_profilenamingprofile) (Audiobookshelf).

Uses POSIX separator rules for creation. Reconcile probes this layout with
wildcard sanitization so historical Windows/S3 keys still match.

```rust
fn default_storage_key(authors: Option<&str>, title: &str, asin: &str, ext: &str) -> String
```



## bookclerk_acquire::naming::resolve_templates

*Function*

Resolve optional template overrides against a naming profile.

```rust
fn resolve_templates(profile: bookclerk_config::NamingProfile, folder_template: Option<&str>, file_template: Option<&str>, chapter_file_template: Option<&str>) -> bookclerk_config::ResolvedNamingTemplates
```



## bookclerk_acquire::naming::sidecar_key

*Function*

Replace the audio file extension on a storage key with a sidecar suffix.

```rust
fn sidecar_key(audio_storage_key: &str, suffix: &str) -> String
```



## bookclerk_acquire::naming::storage_key

*Function*

Build a storage key from optional classic Libation folder/file templates.

```rust
fn storage_key(ctx: &NamingContext, folder_template: Option<&str>, file_template: Option<&str>, ext: &str) -> String
```



## bookclerk_acquire::naming::storage_key_with_contexts

*Function*

Build a storage key using separate folder and file naming contexts.

Used when `SavePodcastsToParentFolder` evaluates the folder template against
the podcast parent while the file template still uses the episode.

```rust
fn storage_key_with_contexts(folder_ctx: &NamingContext, file_ctx: &NamingContext, folder_template: Option<&str>, file_template: Option<&str>, ext: &str, replacement_rules: &[bookclerk_config::ReplacementRule], path_limits: bookclerk_config::PathLimits) -> String
```



## bookclerk_acquire::naming::storage_key_with_rules

*Function*

Like [`storage_key`](naming.md#bookclerk_acquirenamingstorage_key) but applies classic `ReplacementCharacters`.

```rust
fn storage_key_with_rules(ctx: &NamingContext, folder_template: Option<&str>, file_template: Option<&str>, ext: &str, replacement_rules: &[bookclerk_config::ReplacementRule]) -> String
```



## bookclerk_acquire::naming::swap_audio_extension

*Function*

Replace the audio file extension on a storage key (e.g. `.m4b` → `.mp3`).

```rust
fn swap_audio_extension(key: &str, new_ext: &str) -> String
```



