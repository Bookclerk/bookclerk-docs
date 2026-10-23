**bookclerk_acquire > match_storage**

# Module: match_storage

## Contents

**Structs**

- [`MatchStorageOptions`](#matchstorageoptions) - Options for [`match_storage_to_library`](match_storage.md#bookclerk_acquirematch_storagematch_storage_to_library).
- [`MatchStorageSummary`](#matchstoragesummary) - Summary of a storage→library match run.

**Functions**

- [`match_storage_to_library`](#match_storage_to_library) - List audio in storage, probe metadata (no body download), match to library.
- [`match_storage_using_index`](#match_storage_using_index) - Match library rows against an existing scan index.

---

## bookclerk_acquire::match_storage::MatchStorageOptions

*Struct*

Options for [`match_storage_to_library`](match_storage.md#bookclerk_acquirematch_storagematch_storage_to_library).

**Fields:**
- `account: Option<String>` - Optional account id filter; `None` means all accounts.
- `clear_missing: bool` - Clear Acquired when no matching audio is found.
- `asins: Vec<String>` - Limit to these ASINs / ids (empty = all).
- `only_mark_found: bool` - When true, only mark found files as Acquired (do not clear missing).
- `only_clear_missing: bool` - When true, only clear Acquired rows with no matching file.
- `fix_layout: bool` - Relocate matched audio (and sidecars) onto the configured template layout.
- `download: bookclerk_source::DownloadOptions` - When true, treat unmatched titles as download candidates.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> MatchStorageOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::match_storage::MatchStorageSummary

*Struct*

Summary of a storage→library match run.

**Fields:**
- `matched: u32` - Titles matched to existing storage objects.
- `relocated: u32` - Titles whose storage_key was updated to a newly matched object.
- `cleared: u32` - Titles cleared from Acquired because no matching file was found.
- `unchanged: u32` - Titles already consistent with storage.
- `unmatched_files: u32` - Storage keys that could not be matched to a library row.

**Trait Implementations:**

- **Default**
  - `fn default() -> MatchStorageSummary`
- **Clone**
  - `fn clone(self: &Self) -> MatchStorageSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_acquire::match_storage::match_storage_to_library

*Function*

List audio in storage, probe metadata (no body download), match to library.

# Errors

Returns an error when the operation fails.

```rust
fn match_storage_to_library(library: &bookclerk_library::LibraryStore, storage: &dyn StorageBackend, options: MatchStorageOptions) -> crate::error::Result<MatchStorageSummary>
```



## bookclerk_acquire::match_storage::match_storage_using_index

*Function*

Match library rows against an existing scan index.

Does not delete the generation. The caller finishes it when the job completes.

# Errors

Returns an error when the operation fails.

```rust
fn match_storage_using_index(library: &bookclerk_library::LibraryStore, storage: &dyn StorageBackend, options: MatchStorageOptions, index: &crate::reconcile::StorageIndex) -> crate::error::Result<MatchStorageSummary>
```



