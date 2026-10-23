**bookclerk_acquire > reconcile**

# Module: reconcile

## Contents

**Structs**

- [`ReconcileOptions`](#reconcileoptions) - Options for [`reconcile_library`](reconcile.md#bookclerk_acquirereconcilereconcile_library).
- [`ReconcileSummary`](#reconcilesummary) - Summary of a reconcile run.
- [`StorageIndex`](#storageindex) - Index of storage object keys, keyed by ASIN found in the path.

**Functions**

- [`extract_asins_from_key`](#extract_asins_from_key) - Extract Audible-like ASINs and ISBN-13 tokens from a storage key / file path.
- [`find_existing_for_book`](#find_existing_for_book) - Find an existing acquired file for a book (planned path or ASIN-in-path).
- [`find_existing_for_request`](#find_existing_for_request) - Same as [`find_existing_for_book`](reconcile.md#bookclerk_acquirereconcilefind_existing_for_book) but for a acquire request before DB status is Acquired.
- [`reconcile_library`](#reconcile_library) - Scan storage and update library acquire status / storage_key for matches.

---

## bookclerk_acquire::reconcile::ReconcileOptions

*Struct*

Options for [`reconcile_library`](reconcile.md#bookclerk_acquirereconcilereconcile_library).

**Fields:**
- `account: Option<String>` - Optional account id filter; `None` means all accounts.
- `clear_missing: bool` - When true, books marked Acquired whose file is missing become NotAcquired.
- `asins: Vec<String>` - Limit to these ASINs (empty = all).
- `only_mark_found: bool` - When true, only mark found files as Acquired (do not clear missing).
- `only_clear_missing: bool` - When true, only clear Acquired rows with no matching file.
- `download: bookclerk_source::DownloadOptions` - Naming prefs (templates, podcast parent folder) for planned-path matching.
- `fence: Option<bookclerk_library::JobFence>` - Lease fence when this reconcile is a phase of a durable job.
- `checkpoint: Option<bookclerk_plugin_abi::JobCheckpoint>` - Checkpoint to resume, when `fence` is set.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ReconcileOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::reconcile::ReconcileSummary

*Struct*

Summary of a reconcile run.

**Fields:**
- `matched: u32` - Titles matched to existing storage objects.
- `cleared: u32` - Titles cleared from Acquired because no matching file was found.
- `unchanged: u32` - Titles already consistent with storage.

**Trait Implementations:**

- **Default**
  - `fn default() -> ReconcileSummary`
- **Clone**
  - `fn clone(self: &Self) -> ReconcileSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::reconcile::StorageIndex

*Struct*

Index of storage object keys, keyed by ASIN found in the path.

**Methods:**

- `fn from_storage(storage: &dyn StorageBackend) -> Result<Self>` - Empty overlay.
- `fn with_scan<impl Into<String>>(scan_id: impl Trait) -> Self` - Overlay plus a durable scan id.
- `fn scan_id(self: &Self) -> Option<&str>` - Durable scan generation, when present.
- `fn insert_key(self: & mut Self, key: String)` - Insert a storage key into the index (test helper / incremental).
- `fn contains_key(self: &Self, key: &str) -> bool` - Returns true when `key` is present in the index.
- `fn keys_matching_pattern(self: &Self, pattern: &str) -> Vec<String>` - Overlay keys matching a reconcile wildcard `pattern`, best media rank first.
- `fn find_key_matching_pattern(self: &Self, pattern: &str) -> Option<&str>` - First key matching a reconcile wildcard `pattern`, preferring better media types.
- `fn best_key_for_asin(self: &Self, asin: &str) -> Option<&str>` - Best matching storage key for an ASIN / ISBN token, if any.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> StorageIndex`
- **Default**
  - `fn default() -> StorageIndex`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::reconcile::extract_asins_from_key

*Function*

Extract Audible-like ASINs and ISBN-13 tokens from a storage key / file path.

Matches:
- filename stem `B00EXAMPLE.m4b`
- bracket form `Title [B00EXAMPLE].m4b` (classic Libation)
- path segment containing a standalone ASIN token
- ISBN-13 shaped tokens (13 digits, optional hyphens)

```rust
fn extract_asins_from_key(key: &str) -> Vec<String>
```



## bookclerk_acquire::reconcile::find_existing_for_book

*Function*

Find an existing acquired file for a book (planned path or ASIN-in-path).

Honors configured folder/file templates and `save_podcasts_to_parent_folder`
via the same path planner as acquire.

# Errors

Returns a storage or database error from the lookup. A completed miss is
`Ok(None)` and is not an error.

```rust
fn find_existing_for_book(index: &StorageIndex, storage: &dyn StorageBackend, library: &bookclerk_library::LibraryStore, book: &bookclerk_library::BookRecord, download: &bookclerk_source::DownloadOptions) -> crate::error::Result<Option<String>>
```



## bookclerk_acquire::reconcile::find_existing_for_request

*Function*

Same as [`find_existing_for_book`](reconcile.md#bookclerk_acquirereconcilefind_existing_for_book) but for a acquire request before DB status is Acquired.

Matching strategy:
1. Exact planned path under the *creation* replacement rules
2. Same templates with sanitizable characters as wildcards (cross OS/backend)
3. Template path without podcast-parent rewrite (exact, then wildcard)
4. ASIN token found anywhere in a storage key

# Errors

Returns a storage or database error from the lookup. A completed miss is
`Ok(None)`.

```rust
fn find_existing_for_request(index: &StorageIndex, storage: &dyn StorageBackend, library: &bookclerk_library::LibraryStore, req: &crate::pipeline::AcquireRequest) -> crate::error::Result<Option<String>>
```



## bookclerk_acquire::reconcile::reconcile_library

*Function*

Scan storage and update library acquire status / storage_key for matches.

# Errors

Returns an error when the operation fails.

```rust
fn reconcile_library(library: &bookclerk_library::LibraryStore, storage: &dyn StorageBackend, options: ReconcileOptions) -> crate::error::Result<ReconcileSummary>
```



