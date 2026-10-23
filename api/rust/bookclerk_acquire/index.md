# bookclerk_acquire

Acquire orchestration: license → download → decrypt → metadata → storage.

Host-facing pipeline for fetching a storefront title, packaging audio, and
writing it to one or more output destinations. DRM decrypt happens inside
content-source plugins; this crate never sees license keys. See
`docs/media.md` for encode/remux details.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ConvertRequest`](#convertrequest) | struct | Options for `convert_book`. |
| [`ConvertSummary`](#convertsummary) | struct | Summary of a batch convert run. |
| [`AcquireDestination`](#acquiredestination) | struct | One configured output destination for an acquire run. |
| [`AcquireDestinations`](#acquiredestinations) | struct | Ordered set of acquire destinations built from the deployed destination plugins. |
| [`MatchStorageOptions`](#matchstorageoptions) | struct | Options for `match_storage_to_library`. |
| [`MatchStorageSummary`](#matchstoragesummary) | struct | Summary of a storage→library match run. |
| [`NamingContext`](#namingcontext) | struct | Metadata available to naming templates. |
| [`AcquireRequest`](#acquirerequest) | struct | Request to acquire a single title. |
| [`AcquireResult`](#acquireresult) | struct | Result after a successful acquire. |
| [`PlannedTitleNames`](#plannedtitlenames) | struct | The names an acquire of one title writes on one destination, as the owner plans them, to bound a host's upload of that title. |
| [`ReconcileOptions`](#reconcileoptions) | struct | Options for `reconcile_library`. |
| [`ReconcileSummary`](#reconcilesummary) | struct | Summary of a reconcile run. |
| [`StorageIndex`](#storageindex) | struct | Index of storage object keys, keyed by ASIN found in the path. |
| [`ScanCheckpoint`](#scancheckpoint) | struct | Versioned scan progress stored in the job payload checkpoint. |
| [`AcquireError`](#acquireerror) | enum | Failures from the acquire / convert / reconcile pipeline. |
| [`convert_book`](#convert_book) | function | Convert one acquired m4b/m4a to mp3 and update the library storage key. |
| [`destination_id`](#destination_id) | function | The id of the output destination of `kind`, as plans, upload manifests and `title_files` name it: its lowercase kind name (a node deploys at most one destination per kind). |
| [`match_storage_to_library`](#match_storage_to_library) | function | List audio in storage, probe metadata (no body download), match to library. |
| [`match_storage_using_index`](#match_storage_using_index) | function | Match library rows against an existing scan index. |
| [`audio_basename`](#audio_basename) | function | Basename of the acquired audio file (for `.cue` `FILE` lines). |
| [`chapter_storage_key`](#chapter_storage_key) | function | Storage key for a split chapter file. |
| [`chapter_storage_key_with_folder`](#chapter_storage_key_with_folder) | function | Chapter storage key with a separate folder naming context (podcast parent folder). |
| [`default_storage_key`](#default_storage_key) | function | Build a relative storage key for a acquired title using the default `NamingProfile` (Audiobookshelf). |
| [`resolve_templates`](#resolve_templates) | function | Resolve optional template overrides against a naming profile. |
| [`sidecar_key`](#sidecar_key) | function | Replace the audio file extension on a storage key with a sidecar suffix. |
| [`storage_key`](#storage_key) | function | Build a storage key from optional classic Libation folder/file templates. |
| [`storage_key_with_contexts`](#storage_key_with_contexts) | function | Build a storage key using separate folder and file naming contexts. |
| [`storage_key_with_rules`](#storage_key_with_rules) | function | Like `storage_key` but applies classic `ReplacementCharacters`. |
| [`swap_audio_extension`](#swap_audio_extension) | function | Replace the audio file extension on a storage key (e.g. `.m4b` → `.mp3`). |
| [`acquire_book`](#acquire_book) | function | Run the acquire pipeline for one book. |
| [`acquire_book_indexed`](#acquire_book_indexed) | function | Acquire with an optional pre-built `StorageIndex` (avoids re-listing storage when liberating many titles). On success, newly written keys are inserted into the index so later books in the same batch can match them. |
| [`acquire_pdf_only`](#acquire_pdf_only) | function | Download and store companion PDF only (classic `acquire --pdf`). |
| [`planned_storage_key`](#planned_storage_key) | function | Compute the storage key that would be used (for dry-run / set-status). |
| [`planned_storage_key_for`](#planned_storage_key_for) | function | Planned audio storage key for `ext`, honoring folder/file templates and `save_podcasts_to_parent_folder`. |
| [`planned_storage_key_with_rules`](#planned_storage_key_with_rules) | function | Like `planned_storage_key_for` but with an explicit replacement-rule set (used by reconcile to probe wildcard patterns across sanitization profiles). |
| [`planned_title_folders`](#planned_title_folders) | function | The folders an acquire of `req` writes the title's files under on a destination with `req.options`: the folder of its planned audio key, which its sidecars, chapter files and parts share. When podcast episodes are saved to the parent's folder, the episode's own folder is included too, since a library without the parent's row names it so. An empty string is the destination's root (a template with no folder). |
| [`planned_title_names`](#planned_title_names) | function | The title's planned names on the destination `req.options` describe: `PlannedTitleNames`. |
| [`extract_asins_from_key`](#extract_asins_from_key) | function | Extract Audible-like ASINs and ISBN-13 tokens from a storage key / file path. |
| [`find_existing_for_book`](#find_existing_for_book) | function | Find an existing acquired file for a book (planned path or ASIN-in-path). |
| [`find_existing_for_request`](#find_existing_for_request) | function | Same as `find_existing_for_book` but for a acquire request before DB status is Acquired. |
| [`reconcile_library`](#reconcile_library) | function | Scan storage and update library acquire status / storage_key for matches. |
| [`scan_storage`](#scan_storage) | function | Builds or resumes a durable identity/object index. |
| [`Result`](#result) | type alias | Result alias for `AcquireError`. |
| [`NAME_WILDCARD`](#name_wildcard) | constant | In a `PlannedTitleNames` pattern: one or more characters of a file name (no `/`), where the source's part or chapter title goes; the marker the owner's upload scope matches (`bookclerk_library::control_plane::TitleNames`). |
| [`NUMBER_WILDCARD`](#number_wildcard) | constant | In a `PlannedTitleNames` pattern: one or more ASCII digits, where a part or chapter number goes (the owner's upload scope matches it too). |

## Structs

### ConvertRequest

```rust
pub struct ConvertRequest {
    pub cache_dir: PathBuf,
    pub force: bool,
    pub lame: LameConfig,
    pub max_sample_rate: Option<u32>,
    pub job_id: Option<String>,
    pub temp_quota_bytes: Option<u64>,
    pub cancel: Option<Arc<AtomicBool>>,
}
```

Options for [`convert_book`](#convert_book).

**Fields**

- `cache_dir`: Scratch directory for temporary acquire/convert files.
- `force`: When true, re-download or re-convert even if output exists.
- `lame`: LAME MP3 encoder settings from config.
- `max_sample_rate`: Optional ceiling on output sample rate in Hz.
- `job_id`: Durable job that owns the scratch reservation, when this convert is queued.
- `temp_quota_bytes`: Byte budget for this conversion's input and output files together.
  
  `None` does not impose a quota. The advertised source size is not the quota:
  bytes are counted as they are written, and encoder output is counted before
  the MP3 is published.
- `cancel`: Set when the caller wants the copy loop to stop.

Implements: `Clone`, `Debug`

### ConvertSummary

```rust
pub struct ConvertSummary {
    pub converted: u32,
    pub skipped: u32,
    pub failed: u32,
}
```

Summary of a batch convert run.

**Fields**

- `converted`: Titles successfully converted in this run.
- `skipped`: Titles skipped (already done or ineligible).
- `failed`: Titles that failed conversion or matching.

Implements: `Clone`, `Debug`, `Default`

### AcquireDestination

```rust
pub struct AcquireDestination {
    pub kind: OutputBackendKind,
    pub backend: Box<dyn StorageBackend>,
    pub options: DownloadOptions,
}
```

One configured output destination for an acquire run.

**Fields**

- `kind`: Destination backend kind (`local`, `s3`, …).
- `backend`: Constructed storage backend for this destination.
- `options`: Source download options (quality, chapter prefs, …).

### AcquireDestinations

```rust
pub struct AcquireDestinations {
    pub items: Vec<AcquireDestination>,
    pub primary: OutputBackendKind,
    pub multi_destination: MultiDestinationMode,
}
```

Ordered set of acquire destinations built from the deployed destination plugins.

**Fields**

- `items`: Configured acquire destinations in evaluation order.
- `primary`: Index of the primary destination used for library status keys.
- `multi_destination`: Whether multiple destinations are enabled for this run.

#### AcquireDestinations::len

```rust
pub fn len(&self) -> usize
```

Number of destinations.

#### AcquireDestinations::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Returns true when no destinations are deployed.

#### AcquireDestinations::primary_destination

```rust
pub fn primary_destination(&self) -> &AcquireDestination
```

Returns the primary destination used for library storage keys.

#### AcquireDestinations::destination

```rust
pub fn destination(&self, kind: OutputBackendKind) -> Option<&AcquireDestination>
```

Returns the destination whose kind equals `kind`, if configured.

#### AcquireDestinations::listing_backend

```rust
pub fn listing_backend(&self) -> Result<Box<dyn StorageBackend>>
```

Clone destination backends into a listing/match storage handle.

Reuses the backends acquire already built instead of spawning the
destination guests a second time.

##### Errors

Returns an error when the operation fails.

#### AcquireDestinations::into_listing_backend

```rust
pub fn into_listing_backend(self) -> Result<Box<dyn StorageBackend>>
```

Consumes this set and returns a listing backend over the primary destination.

##### Errors

Returns an error when the operation fails.

### MatchStorageOptions

```rust
pub struct MatchStorageOptions {
    pub account: Option<String>,
    pub clear_missing: bool,
    pub asins: Vec<String>,
    pub only_mark_found: bool,
    pub only_clear_missing: bool,
    pub fix_layout: bool,
    pub download: DownloadOptions,
}
```

Options for [`match_storage_to_library`](#match_storage_to_library).

**Fields**

- `account`: Optional account id filter; `None` means all accounts.
- `clear_missing`: Clear Acquired when no matching audio is found.
- `asins`: Limit to these ASINs / ids (empty = all).
- `only_mark_found`: When true, only mark found files as Acquired (do not clear missing).
- `only_clear_missing`: When true, only clear Acquired rows with no matching file.
- `fix_layout`: Relocate matched audio (and sidecars) onto the configured template layout.
- `download`: When true, treat unmatched titles as download candidates.

Implements: `Clone`, `Debug`, `Default`

### MatchStorageSummary

```rust
pub struct MatchStorageSummary {
    pub matched: u32,
    pub relocated: u32,
    pub cleared: u32,
    pub unchanged: u32,
    pub unmatched_files: u32,
}
```

Summary of a storage→library match run.

**Fields**

- `matched`: Titles matched to existing storage objects.
- `relocated`: Titles whose storage_key was updated to a newly matched object.
- `cleared`: Titles cleared from Acquired because no matching file was found.
- `unchanged`: Titles already consistent with storage.
- `unmatched_files`: Storage keys that could not be matched to a library row.

Implements: `Clone`, `Debug`, `Default`

### NamingContext

```rust
pub struct NamingContext {
    pub asin: String,
    pub title: String,
    pub subtitle: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub series_asin: Option<String>,
    pub year_published: Option<i32>,
    pub account_id: Option<String>,
    pub account_nickname: Option<String>,
    pub locale: Option<String>,
    pub language: Option<String>,
    pub publisher: Option<String>,
    pub categories: Option<String>,
    pub length_minutes: Option<i64>,
    pub bitrate: Option<i64>,
    pub samplerate: Option<i64>,
    pub channels: Option<i64>,
    pub codec: Option<String>,
    pub is_abridged: bool,
    pub content_kind: Option<String>,
    pub chapter_number: Option<u32>,
    pub chapter_count: Option<u32>,
    pub chapter_title: Option<String>,
}
```

Metadata available to naming templates.

**Fields**

- `asin`: Primary product id (Audible ASIN, ISBN, …) for this job.
- `title`: Display title used for naming and metadata.
- `subtitle`: Optional subtitle substituted into naming templates.
- `authors`: Author list for naming templates and tags.
- `narrators`: Narrator list for naming templates and tags.
- `series`: Series name for naming templates, when present.
- `series_index`: Series position string for naming templates, when present.
- `series_asin`: Podcast parent / series ASIN for `SavePodcastsToParentFolder`.
- `year_published`: Publication year for `<year>` (from `published_at` when known).
- `account_id`: Store account id that owns this title.
- `account_nickname`: Operator-facing account label for `<account nickname>` templates.
- `locale`: Marketplace / locale code for naming templates.
- `language`: Content language code for naming templates.
- `publisher`: Publisher name for naming templates, when known.
- `categories`: Category / genre path list for naming templates.
- `length_minutes`: Runtime in minutes for naming templates, when known.
- `bitrate`: Audio bitrate label substituted into naming templates, when known.
- `samplerate`: Audio sample rate in Hz for naming templates, when known.
- `channels`: Channel count for naming templates, when known.
- `codec`: Audio codec label for naming templates, when known.
- `is_abridged`: Whether the edition is abridged (template boolean).
- `content_kind`: Title kind (`book`, `episode`, …) for naming templates.
- `chapter_number`: Chapter-specific: 1-based index.
- `chapter_count`: Total chapter count when naming a split chapter file.
- `chapter_title`: Chapter title when naming a split chapter file.

Implements: `Clone`, `Debug`, `Default`

### AcquireRequest

```rust
pub struct AcquireRequest {
    pub asin: String,
    pub book_uuid: Option<String>,
    pub source: String,
    pub account_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub options: DownloadOptions,
    pub files_dir: PathBuf,
    pub cache_dir: PathBuf,
    pub force: bool,
    pub write_destinations: Option<Vec<OutputBackendKind>>,
    pub job_id: Option<String>,
    pub temp_quota_bytes: Option<u64>,
}
```

Request to acquire a single title.

**Fields**

- `asin`: Download product id (Audible ASIN / Libro ISBN).
- `book_uuid`: Stable library UUID when known (preferred for status updates).
- `source`: Which store owns this title (plugin id: `audible`, `libro`, …).
- `account_id`: Store account id that owns this title.
- `title`: Display title used for naming and metadata.
- `authors`: Author list for naming templates and tags.
- `narrators`: Narrator list for naming templates and tags.
- `series`: Series name for naming templates, when present.
- `series_index`: Series position string for naming templates, when present.
- `options`: Source download options (quality, chapter prefs, …).
- `files_dir`: Bookclerk files directory root (`BOOKCLERK_FILES_DIR`).
- `cache_dir`: Scratch directory for encrypted + decrypted temps.
- `force`: When true, download even if matching media already exists in storage.
- `write_destinations`: When set, only write prepared audio to these destination kinds
  (`output.multi_destination = refetch_missing`).
- `job_id`: Durable daemon job id that owns this acquire's scratch directory.
- `temp_quota_bytes`: Refuse a new scratch dir when registered temps already exceed this.

Implements: `Clone`, `Debug`

### AcquireResult

```rust
pub struct AcquireResult {
    pub asin: String,
    pub storage_key: String,
    pub written_keys: Vec<String>,
    pub matched_existing: bool,
}
```

Result after a successful acquire.

**Fields**

- `asin`: Primary product id (Audible ASIN, ISBN, …) for this job.
- `storage_key`: Object-storage key written for the primary audio artifact.
- `written_keys`: All object keys written during this acquire (audio + sidecars).
- `matched_existing`: True when an existing file was matched and no download ran.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### PlannedTitleNames

```rust
pub struct PlannedTitleNames {
    pub folders: Vec<String>,
    pub own_folder: bool,
    pub audio_stems: Vec<String>,
    pub part_patterns: Vec<String>,
}
```

The names an acquire of one title writes on one destination, as the
owner plans them, to bound a host's upload of that title.

**Fields**

- `folders`: The folders the title's files go under ([`planned_title_folders`](#planned_title_folders)).
- `own_folder`: True when the folder template has a component of the title's own
  (its id, title or subtitle), so no other title's files go there and
  any name under it is the title's. False for a folder several titles
  share (`<author>`), the destination's root, or a podcast parent's
  folder: there only the names below are the title's.
- `audio_stems`: The planned audio key of a single-file acquire, without its
  extension (one per folder variant): the title's audio is this stem
  with any audio extension, and its sidecars this stem, a `.` and a
  suffix.
- `part_patterns`: Part and chapter keys without their extension, with
  [`NAME_WILDCARD`](#name_wildcard) and [`NUMBER_WILDCARD`](#number_wildcard) where the source's part or
  chapter title and number go. Those titles are not known before the
  acquire runs, so they are matched rather than listed; a template
  with no part or chapter field yields a literal name.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ReconcileOptions

```rust
pub struct ReconcileOptions {
    pub account: Option<String>,
    pub clear_missing: bool,
    pub asins: Vec<String>,
    pub only_mark_found: bool,
    pub only_clear_missing: bool,
    pub download: DownloadOptions,
    pub fence: Option<JobFence>,
    pub checkpoint: Option<JobCheckpoint>,
}
```

Options for [`reconcile_library`](#reconcile_library).

**Fields**

- `account`: Optional account id filter; `None` means all accounts.
- `clear_missing`: When true, books marked Acquired whose file is missing become NotAcquired.
- `asins`: Limit to these ASINs (empty = all).
- `only_mark_found`: When true, only mark found files as Acquired (do not clear missing).
- `only_clear_missing`: When true, only clear Acquired rows with no matching file.
- `download`: Naming prefs (templates, podcast parent folder) for planned-path matching.
- `fence`: Lease fence when this reconcile is a phase of a durable job.
- `checkpoint`: Checkpoint to resume, when `fence` is set.

Implements: `Clone`, `Debug`, `Default`

### ReconcileSummary

```rust
pub struct ReconcileSummary {
    pub matched: u32,
    pub cleared: u32,
    pub unchanged: u32,
}
```

Summary of a reconcile run.

**Fields**

- `matched`: Titles matched to existing storage objects.
- `cleared`: Titles cleared from Acquired because no matching file was found.
- `unchanged`: Titles already consistent with storage.

Implements: `Clone`, `Debug`, `Default`

### StorageIndex

```rust
pub struct StorageIndex {
    /* private fields */
}
```

Index of storage object keys, keyed by ASIN found in the path.

#### StorageIndex::from_storage

```rust
pub async fn from_storage(storage: &dyn StorageBackend) -> Result<Self>
```

Empty overlay.

This does not list `storage`. Call [`crate::scan_storage`](#scan_storage)
when identity matching needs a durable index. Exact keys are resolved
with `StorageBackend::exists` at decision time.

##### Errors

Always returns `Ok`. The storage argument is accepted so older call
sites keep compiling while they move to [`crate::scan_storage`](#scan_storage).

#### StorageIndex::with_scan

```rust
pub fn with_scan(scan_id: impl Into<String>) -> Self
```

Overlay plus a durable scan id.

#### StorageIndex::scan_id

```rust
pub fn scan_id(&self) -> Option<&str>
```

Durable scan generation, when present.

#### StorageIndex::insert_key

```rust
pub fn insert_key(&mut self, key: String)
```

Insert a storage key into the index (test helper / incremental).

#### StorageIndex::contains_key

```rust
pub fn contains_key(&self, key: &str) -> bool
```

Returns true when `key` is present in the index.

#### StorageIndex::keys_matching_pattern

```rust
pub fn keys_matching_pattern(&self, pattern: &str) -> Vec<String>
```

Overlay keys matching a reconcile wildcard `pattern`, best media rank first.

#### StorageIndex::find_key_matching_pattern

```rust
pub fn find_key_matching_pattern(&self, pattern: &str) -> Option<&str>
```

First key matching a reconcile wildcard `pattern`, preferring better media types.

#### StorageIndex::best_key_for_asin

```rust
pub fn best_key_for_asin(&self, asin: &str) -> Option<&str>
```

Best matching storage key for an ASIN / ISBN token, if any.

Implements: `Clone`, `Debug`, `Default`

### ScanCheckpoint

```rust
pub struct ScanCheckpoint {
    pub v: u32,
    pub op: String,
    pub instance_id: String,
    pub namespace: String,
    pub generation: String,
    pub phase: String,
    pub cursor: Option<String>,
    pub node_local: bool,
    pub placement: Option<String>,
}
```

Versioned scan progress stored in the job payload checkpoint.

**Fields**

- `v`: Schema version. Unknown versions fail closed.
- `op`: Always `storage_scan`.
- `instance_id`: `StorageBackend::instance_id` this cursor belongs to.
- `namespace`: List prefix (empty scans the namespace).
- `generation`: `storage_scan_rows.scan_id`.
- `phase`: `list` while paging, `apply` after the scan committed.
- `cursor`: Backend list cursor. `None` starts at the first page.
- `node_local`: True when the list index or scratch is node-local.
- `placement`: Host placement for node-local bytes. Absent means the scan was recorded
  as portable. A mismatch restarts instead of adopting another node's rows.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### AcquireError

```rust
pub enum AcquireError {
    Source(SourceError),
    Storage(StorageError),
    Media(MediaError),
    Library(LibraryError),
    Io(Error),
    Other(Error),
}
```

Failures from the acquire / convert / reconcile pipeline.

**Variants**

- `Source`: Failure originating in a content-source plugin.
- `Storage`: Failure originating in an object-storage backend.
- `Media`: Failure originating in media encode/remux/fixup.
- `Library`: Failure originating in the library database layer.
- `Io`: Filesystem or other I/O failure.
- `Other`: Catch-all for otherwise unclassified failures.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<LibraryError>`, `From<MediaError>`, `From<SourceError>`, `From<StorageError>`

## Functions

### convert_book

```rust
pub async fn convert_book(
    library: &LibraryStore,
    storage: &dyn StorageBackend,
    book: &BookRecord,
    req: &ConvertRequest,
) -> Result<String>
```

Convert one acquired m4b/m4a to mp3 and update the library storage key.

#### Arguments

* `library` - Library store used to update acquire status / storage key.
* `storage` - Object storage backend holding the source and destination objects.
* `book` - Acquired book row whose `storage_key` points at m4b/m4a.
* `req` - Cache directory, force flag, and LAME settings.

#### Returns

Object-storage key of the written MP3.

#### Errors

Returns [`AcquireError`](#acquireerror) when the source is missing/ineligible, encode fails,
or library/storage updates fail.

### destination_id

```rust
pub fn destination_id(kind: OutputBackendKind) -> &'static str
```

The id of the output destination of `kind`, as plans, upload manifests
and `title_files` name it: its lowercase kind name (a node deploys at
most one destination per kind).

### match_storage_to_library

```rust
pub async fn match_storage_to_library(
    library: &LibraryStore,
    storage: &dyn StorageBackend,
    options: MatchStorageOptions,
) -> Result<MatchStorageSummary>
```

List audio in storage, probe metadata (no body download), match to library.

#### Errors

Returns an error when the operation fails.

### match_storage_using_index

```rust
pub async fn match_storage_using_index(
    library: &LibraryStore,
    storage: &dyn StorageBackend,
    options: MatchStorageOptions,
    index: &StorageIndex,
) -> Result<MatchStorageSummary>
```

Match library rows against an existing scan index.

Does not delete the generation. The caller finishes it when the job completes.

#### Errors

Returns an error when the operation fails.

### audio_basename

```rust
pub fn audio_basename(storage_key: &str) -> String
```

Basename of the acquired audio file (for `.cue` `FILE` lines).

### chapter_storage_key

```rust
pub fn chapter_storage_key(
    ctx: &NamingContext,
    folder_template: Option<&str>,
    chapter_file_template: Option<&str>,
    replacement_rules: &[ReplacementRule],
    chapter_number: usize,
    chapter_title: &str,
    ext: &str,
) -> String
```

Storage key for a split chapter file.

### chapter_storage_key_with_folder

```rust
pub fn chapter_storage_key_with_folder(
    folder_ctx: &NamingContext,
    file_ctx: &NamingContext,
    folder_template: Option<&str>,
    chapter_file_template: Option<&str>,
    replacement_rules: &[ReplacementRule],
    chapter_number: usize,
    chapter_title: &str,
    ext: &str,
    path_limits: PathLimits,
) -> String
```

Chapter storage key with a separate folder naming context (podcast parent folder).

### default_storage_key

```rust
pub fn default_storage_key(
    authors: Option<&str>,
    title: &str,
    asin: &str,
    ext: &str,
) -> String
```

Build a relative storage key for a acquired title using the default
[`NamingProfile`](../bookclerk_config/index.md#namingprofile) (Audiobookshelf).

Uses POSIX separator rules for creation. Reconcile probes this layout with
wildcard sanitization so historical Windows/S3 keys still match.

### resolve_templates

```rust
pub fn resolve_templates(
    profile: NamingProfile,
    folder_template: Option<&str>,
    file_template: Option<&str>,
    chapter_file_template: Option<&str>,
) -> ResolvedNamingTemplates
```

Resolve optional template overrides against a naming profile.

### sidecar_key

```rust
pub fn sidecar_key(audio_storage_key: &str, suffix: &str) -> String
```

Replace the audio file extension on a storage key with a sidecar suffix.

### storage_key

```rust
pub fn storage_key(
    ctx: &NamingContext,
    folder_template: Option<&str>,
    file_template: Option<&str>,
    ext: &str,
) -> String
```

Build a storage key from optional classic Libation folder/file templates.

### storage_key_with_contexts

```rust
pub fn storage_key_with_contexts(
    folder_ctx: &NamingContext,
    file_ctx: &NamingContext,
    folder_template: Option<&str>,
    file_template: Option<&str>,
    ext: &str,
    replacement_rules: &[ReplacementRule],
    path_limits: PathLimits,
) -> String
```

Build a storage key using separate folder and file naming contexts.

Used when `SavePodcastsToParentFolder` evaluates the folder template against
the podcast parent while the file template still uses the episode.

### storage_key_with_rules

```rust
pub fn storage_key_with_rules(
    ctx: &NamingContext,
    folder_template: Option<&str>,
    file_template: Option<&str>,
    ext: &str,
    replacement_rules: &[ReplacementRule],
) -> String
```

Like [`storage_key`](#storage_key) but applies classic `ReplacementCharacters`.

### swap_audio_extension

```rust
pub fn swap_audio_extension(key: &str, new_ext: &str) -> String
```

Replace the audio file extension on a storage key (e.g. `.m4b` → `.mp3`).

### acquire_book

```rust
pub async fn acquire_book(
    library: &LibraryStore,
    destinations: &AcquireDestinations,
    req: AcquireRequest,
    source: &dyn ContentSource,
) -> Result<AcquireResult>
```

Run the acquire pipeline for one book.

#### Arguments

* `library` - Library store for status updates.
* `source` - Content-source plugin used to fetch the title.
* `destinations` - Output backends to write packaged audio into.
* `req` - Title identity, account, and download options.

#### Returns

[`AcquireResult`](#acquireresult) describing written keys and match status.

#### Errors

Returns [`AcquireError`](#acquireerror) when fetch, package, storage, or library updates fail.

### acquire_book_indexed

```rust
pub async fn acquire_book_indexed(
    library: &LibraryStore,
    destinations: &AcquireDestinations,
    req: AcquireRequest,
    index: Option<&mut StorageIndex>,
    source: &dyn ContentSource,
) -> Result<AcquireResult>
```

Acquire with an optional pre-built [`StorageIndex`](#storageindex) (avoids re-listing storage
when liberating many titles). On success, newly written keys are inserted into
the index so later books in the same batch can match them.

Fetch always goes through `ContentSource::fetch_title` (Plain only;
Audible decrypts Adrm/CENC inside the plugin before returning).

Every key the run writes on a destination (audio, chapter or part
files, sidecars, the companion PDF) is recorded for the title in the
library's `title_files` just before it is written, so a host upload of
another title is never granted it.

#### Errors

Returns an error when the operation fails.

### acquire_pdf_only

```rust
pub async fn acquire_pdf_only(
    library: &LibraryStore,
    destinations: &AcquireDestinations,
    req: &AcquireRequest,
    source: &dyn ContentSource,
) -> Result<AcquireResult>
```

Download and store companion PDF only (classic `acquire --pdf`).

Fetches via `ContentSource::fetch_title`; requires `PlainFetch.pdf_url`, then HTTP GETs that URL.
Each PDF key written is recorded for the title in `title_files`.

#### Errors

Returns an error when the operation fails.

### planned_storage_key

```rust
pub async fn planned_storage_key(library: &LibraryStore, req: &AcquireRequest) -> String
```

Compute the storage key that would be used (for dry-run / set-status).

Uses the library row (when present) so podcast episodes honor
`save_podcasts_to_parent_folder` the same way as a real acquire.

### planned_storage_key_for

```rust
pub async fn planned_storage_key_for(
    library: &LibraryStore,
    req: &AcquireRequest,
    ext: &str,
) -> String
```

Planned audio storage key for `ext`, honoring folder/file templates and
`save_podcasts_to_parent_folder`.

### planned_storage_key_with_rules

```rust
pub async fn planned_storage_key_with_rules(
    library: &LibraryStore,
    req: &AcquireRequest,
    ext: &str,
    replacement_rules: &[ReplacementRule],
) -> String
```

Like [`planned_storage_key_for`](#planned_storage_key_for) but with an explicit replacement-rule set
(used by reconcile to probe wildcard patterns across sanitization profiles).

### planned_title_folders

```rust
pub async fn planned_title_folders(
    library: &LibraryStore,
    req: &AcquireRequest,
) -> Vec<String>
```

The folders an acquire of `req` writes the title's files under on a
destination with `req.options`: the folder of its planned audio key,
which its sidecars, chapter files and parts share. When podcast
episodes are saved to the parent's folder, the episode's own folder is
included too, since a library without the parent's row names it so. An
empty string is the destination's root (a template with no folder).

### planned_title_names

```rust
pub async fn planned_title_names(
    library: &LibraryStore,
    req: &AcquireRequest,
) -> PlannedTitleNames
```

The title's planned names on the destination `req.options` describe:
[`PlannedTitleNames`](#plannedtitlenames).

### extract_asins_from_key

```rust
pub fn extract_asins_from_key(key: &str) -> Vec<String>
```

Extract Audible-like ASINs and ISBN-13 tokens from a storage key / file path.

Matches:
- filename stem `B00EXAMPLE.m4b`
- bracket form `Title [B00EXAMPLE].m4b` (classic Libation)
- path segment containing a standalone ASIN token
- ISBN-13 shaped tokens (13 digits, optional hyphens)

### find_existing_for_book

```rust
pub async fn find_existing_for_book(
    index: &StorageIndex,
    storage: &dyn StorageBackend,
    library: &LibraryStore,
    book: &BookRecord,
    download: &DownloadOptions,
) -> Result<Option<String>>
```

Find an existing acquired file for a book (planned path or ASIN-in-path).

Honors configured folder/file templates and `save_podcasts_to_parent_folder`
via the same path planner as acquire.

#### Errors

Returns a storage or database error from the lookup. A completed miss is
`Ok(None)` and is not an error.

### find_existing_for_request

```rust
pub async fn find_existing_for_request(
    index: &StorageIndex,
    storage: &dyn StorageBackend,
    library: &LibraryStore,
    req: &AcquireRequest,
) -> Result<Option<String>>
```

Same as [`find_existing_for_book`](#find_existing_for_book) but for a acquire request before DB status is Acquired.

Matching strategy:
1. Exact planned path under the *creation* replacement rules
2. Same templates with sanitizable characters as wildcards (cross OS/backend)
3. Template path without podcast-parent rewrite (exact, then wildcard)
4. ASIN token found anywhere in a storage key

#### Errors

Returns a storage or database error from the lookup. A completed miss is
`Ok(None)`.

### reconcile_library

```rust
pub async fn reconcile_library(
    library: &LibraryStore,
    storage: &dyn StorageBackend,
    options: ReconcileOptions,
) -> Result<ReconcileSummary>
```

Scan storage and update library acquire status / storage_key for matches.

#### Errors

Returns an error when the operation fails.

### scan_storage

```rust
pub async fn scan_storage(
    library: &LibraryStore,
    storage: &dyn StorageBackend,
    fence: Option<&JobFence>,
    prior: Option<&JobCheckpoint>,
    probe_metadata: bool,
) -> Result<StorageIndex>
```

Builds or resumes a durable identity/object index.

#### Errors

Returns [`AcquireError`](#acquireerror) when listing, the database, or a checkpoint write fails,
or when `prior` names a different storage instance.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, AcquireError>;
```

Result alias for [`AcquireError`](#acquireerror).

## Constants

### NAME_WILDCARD

```rust
pub const NAME_WILDCARD: char = bookclerk_library::control_plane::UPLOAD_NAME_WILDCARD;
```

In a [`PlannedTitleNames`](#plannedtitlenames) pattern: one or more characters of a file
name (no `/`), where the source's part or chapter title goes; the
marker the owner's upload scope matches
([`bookclerk_library::control_plane::TitleNames`](../bookclerk_library/control_plane.md#titlenames)).

### NUMBER_WILDCARD

```rust
pub const NUMBER_WILDCARD: char = bookclerk_library::control_plane::UPLOAD_NUMBER_WILDCARD;
```

In a [`PlannedTitleNames`](#plannedtitlenames) pattern: one or more ASCII digits, where a
part or chapter number goes (the owner's upload scope matches it too).
