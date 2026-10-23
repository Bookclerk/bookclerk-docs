**bookclerk_acquire > pipeline**

# Module: pipeline

## Contents

**Structs**

- [`AcquireRequest`](#acquirerequest) - Request to acquire a single title.
- [`AcquireResult`](#acquireresult) - Result after a successful acquire.
- [`PlannedTitleNames`](#plannedtitlenames) - The names an acquire of one title writes on one destination, as the

**Functions**

- [`acquire_book`](#acquire_book) - Run the acquire pipeline for one book.
- [`acquire_book_indexed`](#acquire_book_indexed) - Acquire with an optional pre-built [`StorageIndex`](reconcile.md#bookclerk_acquirereconcilestorageindex) (avoids re-listing storage
- [`acquire_pdf_only`](#acquire_pdf_only) - Download and store companion PDF only (classic `acquire --pdf`).
- [`planned_storage_key`](#planned_storage_key) - Compute the storage key that would be used (for dry-run / set-status).
- [`planned_storage_key_for`](#planned_storage_key_for) - Planned audio storage key for `ext`, honoring folder/file templates and
- [`planned_storage_key_with_rules`](#planned_storage_key_with_rules) - Like [`planned_storage_key_for`](pipeline.md#bookclerk_acquirepipelineplanned_storage_key_for) but with an explicit replacement-rule set
- [`planned_title_folders`](#planned_title_folders) - The folders an acquire of `req` writes the title's files under on a
- [`planned_title_names`](#planned_title_names) - The title's planned names on the destination `req.options` describe:

**Constants**

- [`NAME_WILDCARD`](#name_wildcard) - In a [`PlannedTitleNames`](pipeline.md#bookclerk_acquirepipelineplannedtitlenames) pattern: one or more characters of a file
- [`NUMBER_WILDCARD`](#number_wildcard) - In a [`PlannedTitleNames`](pipeline.md#bookclerk_acquirepipelineplannedtitlenames) pattern: one or more ASCII digits, where a

---

## bookclerk_acquire::pipeline::AcquireRequest

*Struct*

Request to acquire a single title.

**Fields:**
- `asin: String` - Download product id (Audible ASIN / Libro ISBN).
- `book_uuid: Option<String>` - Stable library UUID when known (preferred for status updates).
- `source: String` - Which store owns this title (plugin id: `audible`, `libro`, …).
- `account_id: String` - Store account id that owns this title.
- `title: String` - Display title used for naming and metadata.
- `authors: Option<String>` - Author list for naming templates and tags.
- `narrators: Option<String>` - Narrator list for naming templates and tags.
- `series: Option<String>` - Series name for naming templates, when present.
- `series_index: Option<String>` - Series position string for naming templates, when present.
- `options: bookclerk_source::DownloadOptions` - Source download options (quality, chapter prefs, …).
- `files_dir: std::path::PathBuf` - Bookclerk files directory root (`BOOKCLERK_FILES_DIR`).
- `cache_dir: std::path::PathBuf` - Scratch directory for encrypted + decrypted temps.
- `force: bool` - When true, download even if matching media already exists in storage.
- `write_destinations: Option<Vec<bookclerk_config::OutputBackendKind>>` - When set, only write prepared audio to these destination kinds
- `job_id: Option<String>` - Durable daemon job id that owns this acquire's scratch directory.
- `temp_quota_bytes: Option<u64>` - Refuse a new scratch dir when registered temps already exceed this.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AcquireRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::pipeline::AcquireResult

*Struct*

Result after a successful acquire.

**Fields:**
- `asin: String` - Primary product id (Audible ASIN, ISBN, …) for this job.
- `storage_key: String` - Object-storage key written for the primary audio artifact.
- `written_keys: Vec<String>` - All object keys written during this acquire (audio + sidecars).
- `matched_existing: bool` - True when an existing file was matched and no download ran.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AcquireResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_acquire::pipeline::NAME_WILDCARD

*Constant*: `char`

In a [`PlannedTitleNames`](pipeline.md#bookclerk_acquirepipelineplannedtitlenames) pattern: one or more characters of a file
name (no `/`), where the source's part or chapter title goes; the
marker the owner's upload scope matches
([`bookclerk_library::control_plane::TitleNames`](../bookclerk_library/control_plane/uploads.md#bookclerk_librarycontrol_planeuploadstitlenames)).



## bookclerk_acquire::pipeline::NUMBER_WILDCARD

*Constant*: `char`

In a [`PlannedTitleNames`](pipeline.md#bookclerk_acquirepipelineplannedtitlenames) pattern: one or more ASCII digits, where a
part or chapter number goes (the owner's upload scope matches it too).



## bookclerk_acquire::pipeline::PlannedTitleNames

*Struct*

The names an acquire of one title writes on one destination, as the
owner plans them, to bound a host's upload of that title.

**Fields:**
- `folders: Vec<String>` - The folders the title's files go under ([`planned_title_folders`](pipeline.md#bookclerk_acquirepipelineplanned_title_folders)).
- `own_folder: bool` - True when the folder template has a component of the title's own
- `audio_stems: Vec<String>` - The planned audio key of a single-file acquire, without its
- `part_patterns: Vec<String>` - Part and chapter keys without their extension, with

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &PlannedTitleNames) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PlannedTitleNames`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::pipeline::acquire_book

*Function*

Run the acquire pipeline for one book.

# Arguments

* `library` - Library store for status updates.
* `source` - Content-source plugin used to fetch the title.
* `destinations` - Output backends to write packaged audio into.
* `req` - Title identity, account, and download options.

# Returns

[`AcquireResult`](pipeline.md#bookclerk_acquirepipelineacquireresult) describing written keys and match status.

# Errors

Returns [`AcquireError`](error.md#bookclerk_acquireerroracquireerror) when fetch, package, storage, or library updates fail.

```rust
fn acquire_book(library: &bookclerk_library::LibraryStore, destinations: &crate::destinations::AcquireDestinations, req: AcquireRequest, source: &dyn ContentSource) -> crate::error::Result<AcquireResult>
```



## bookclerk_acquire::pipeline::acquire_book_indexed

*Function*

Acquire with an optional pre-built [`StorageIndex`](reconcile.md#bookclerk_acquirereconcilestorageindex) (avoids re-listing storage
when liberating many titles). On success, newly written keys are inserted into
the index so later books in the same batch can match them.

Fetch always goes through `ContentSource::fetch_title` (Plain only;
Audible decrypts Adrm/CENC inside the plugin before returning).

Every key the run writes on a destination (audio, chapter or part
files, sidecars, the companion PDF) is recorded for the title in the
library's `title_files` just before it is written, so a host upload of
another title is never granted it.

# Errors

Returns an error when the operation fails.

```rust
fn acquire_book_indexed(library: &bookclerk_library::LibraryStore, destinations: &crate::destinations::AcquireDestinations, req: AcquireRequest, index: Option<& mut crate::reconcile::StorageIndex>, source: &dyn ContentSource) -> crate::error::Result<AcquireResult>
```



## bookclerk_acquire::pipeline::acquire_pdf_only

*Function*

Download and store companion PDF only (classic `acquire --pdf`).

Fetches via `ContentSource::fetch_title`; requires `PlainFetch.pdf_url`, then HTTP GETs that URL.
Each PDF key written is recorded for the title in `title_files`.

# Errors

Returns an error when the operation fails.

```rust
fn acquire_pdf_only(library: &bookclerk_library::LibraryStore, destinations: &crate::destinations::AcquireDestinations, req: &AcquireRequest, source: &dyn ContentSource) -> crate::error::Result<AcquireResult>
```



## bookclerk_acquire::pipeline::planned_storage_key

*Function*

Compute the storage key that would be used (for dry-run / set-status).

Uses the library row (when present) so podcast episodes honor
`save_podcasts_to_parent_folder` the same way as a real acquire.

```rust
fn planned_storage_key(library: &bookclerk_library::LibraryStore, req: &AcquireRequest) -> String
```



## bookclerk_acquire::pipeline::planned_storage_key_for

*Function*

Planned audio storage key for `ext`, honoring folder/file templates and
`save_podcasts_to_parent_folder`.

```rust
fn planned_storage_key_for(library: &bookclerk_library::LibraryStore, req: &AcquireRequest, ext: &str) -> String
```



## bookclerk_acquire::pipeline::planned_storage_key_with_rules

*Function*

Like [`planned_storage_key_for`](pipeline.md#bookclerk_acquirepipelineplanned_storage_key_for) but with an explicit replacement-rule set
(used by reconcile to probe wildcard patterns across sanitization profiles).

```rust
fn planned_storage_key_with_rules(library: &bookclerk_library::LibraryStore, req: &AcquireRequest, ext: &str, replacement_rules: &[bookclerk_config::ReplacementRule]) -> String
```



## bookclerk_acquire::pipeline::planned_title_folders

*Function*

The folders an acquire of `req` writes the title's files under on a
destination with `req.options`: the folder of its planned audio key,
which its sidecars, chapter files and parts share. When podcast
episodes are saved to the parent's folder, the episode's own folder is
included too, since a library without the parent's row names it so. An
empty string is the destination's root (a template with no folder).

```rust
fn planned_title_folders(library: &bookclerk_library::LibraryStore, req: &AcquireRequest) -> Vec<String>
```



## bookclerk_acquire::pipeline::planned_title_names

*Function*

The title's planned names on the destination `req.options` describe:
[`PlannedTitleNames`](pipeline.md#bookclerk_acquirepipelineplannedtitlenames).

```rust
fn planned_title_names(library: &bookclerk_library::LibraryStore, req: &AcquireRequest) -> PlannedTitleNames
```



