**bookclerk_library > control_plane > branch_catalog**

# Module: control_plane::branch_catalog

## Contents

**Structs**

- [`BranchTitle`](#branchtitle) - One title a branch shared, as the owner's Settings → Branches lists it.
- [`BranchTitleCounts`](#branchtitlecounts) - Counts of one branch's shared titles.
- [`CatalogSessionOpened`](#catalogsessionopened) - An opened catalog session ([`begin_catalog`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogbegin_catalog)).

**Enums**

- [`CatalogAccept`](#catalogaccept) - Outcome of [`accept_catalog_batch`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogaccept_catalog_batch).
- [`CatalogBegin`](#catalogbegin) - Outcome of [`begin_catalog`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogbegin_catalog).

**Functions**

- [`accept_catalog_batch`](#accept_catalog_batch) - Records one batch of `node_id`'s shared titles under `fence`, then the
- [`begin_catalog`](#begin_catalog) - Opens (or reopens) `node_id`'s catalog sync with the settings the
- [`branch_provenance`](#branch_provenance) - The provenance text of titles shared by `node_id`.
- [`branch_title_counts`](#branch_title_counts) - [`BranchTitleCounts`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogbranchtitlecounts) of `node_id`.
- [`catalog_fence_live`](#catalog_fence_live) - True while `fence` is `node_id`'s live catalog fence and it may share.
- [`detach_branch_titles`](#detach_branch_titles) - Marks every title `node_id` shares detached with `reason`; answers how
- [`forwardable_branch_titles`](#forwardable_branch_titles) - On a branch: the titles its own branches shared with it that it may
- [`grant_catalog_upload`](#grant_catalog_upload) - Opens an upload of one shared title's files, bound to `node_id`'s
- [`list_branch_titles`](#list_branch_titles) - Titles branches shared, of one branch or all, oldest first, at most
- [`pending_deliveries`](#pending_deliveries) - Up to `limit` of `node_id`'s shared titles whose files have not
- [`remove_detached_branch_titles`](#remove_detached_branch_titles) - Removes detached entries ("from a branch that left" and the like), of

**Constants**

- [`BRANCH_TITLE_DETACHED`](#branch_title_detached) - `branch_titles.state` of a title from a branch that stopped sharing,
- [`BRANCH_TITLE_SHARED`](#branch_title_shared) - `branch_titles.state` of a title its branch still shares.
- [`DETACHED_LEFT`](#detached_left) - Detach reason: the branch left the library (or was revoked).
- [`DETACHED_STOPPED_SHARING`](#detached_stopped_sharing) - Detach reason: the branch turned "Share this branch's books" off.
- [`DETACHED_WITHDRAWN`](#detached_withdrawn) - Detach reason: a full pass of the branch's catalog no longer named it.
- [`FULL_CATALOG_PASS_EVERY`](#full_catalog_pass_every) - How long a branch's catalog sync resumes incrementally before the

---

## bookclerk_library::control_plane::branch_catalog::BRANCH_TITLE_DETACHED

*Constant*: `&str`

`branch_titles.state` of a title from a branch that stopped sharing,
left, or withdrew it.



## bookclerk_library::control_plane::branch_catalog::BRANCH_TITLE_SHARED

*Constant*: `&str`

`branch_titles.state` of a title its branch still shares.



## bookclerk_library::control_plane::branch_catalog::BranchTitle

*Struct*

One title a branch shared, as the owner's Settings → Branches lists it.

**Fields:**
- `uuid: String` - Stable id.
- `provenance: String` - `node:<node_id>`.
- `node_id: String` - The branch's node id.
- `owner_user_id: Option<i64>` - The user it belongs to here (the branch's owner).
- `store: String` - Store id.
- `product_id: String` - Product id.
- `title: String` - Display title.
- `authors: Option<String>` - Authors.
- `primary_key: String` - Storage key of the primary audio file.
- `delivered: bool` - True once the current manifest's files arrived.
- `state: String` - `shared` or `detached`.
- `detached_reason: Option<String>` - `stopped_sharing`, `left` or `withdrawn`.
- `updated_at: String` - RFC 3339 last change.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BranchTitle`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &BranchTitle) -> bool`
- **From**
  - `fn from(row: branch_titles::Model) -> Self`



## bookclerk_library::control_plane::branch_catalog::BranchTitleCounts

*Struct*

Counts of one branch's shared titles.

**Fields:**
- `shared: u64` - Titles it shares now.
- `delivered: u64` - Of those, titles whose files arrived.
- `detached: u64` - Titles marked detached (it stopped sharing, left, or withdrew them).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BranchTitleCounts`
- **PartialEq**
  - `fn eq(self: &Self, other: &BranchTitleCounts) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> BranchTitleCounts`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::branch_catalog::CatalogAccept

*Enum*

Outcome of [`accept_catalog_batch`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogaccept_catalog_batch).

**Variants:**
- `Accepted{ upserted: u64, skipped: u64, wanted: Vec<bookclerk_node_proto::CatalogRef> }` - The batch was recorded.
- `Lost` - The fence is not live (or the branch no longer shares or is no
- `Refused(&'static str)` - The batch is not allowed; nothing was recorded.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogAccept`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogAccept) -> bool`



## bookclerk_library::control_plane::branch_catalog::CatalogBegin

*Enum*

Outcome of [`begin_catalog`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogbegin_catalog).

**Variants:**
- `Opened(CatalogSessionOpened)` - The session is open under a new fence.
- `Refused(&'static str)` - The node is not a branch this library trusts.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogBegin`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogBegin) -> bool`



## bookclerk_library::control_plane::branch_catalog::CatalogSessionOpened

*Struct*

An opened catalog session ([`begin_catalog`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogbegin_catalog)).

**Fields:**
- `fence: i64` - The new catalog fence.
- `cursor: String` - Where the branch resumes (empty: from the start).
- `wanted: Vec<bookclerk_node_proto::CatalogRef>` - Titles recorded earlier whose files have not arrived.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogSessionOpened) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSessionOpened`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::branch_catalog::DETACHED_LEFT

*Constant*: `&str`

Detach reason: the branch left the library (or was revoked).



## bookclerk_library::control_plane::branch_catalog::DETACHED_STOPPED_SHARING

*Constant*: `&str`

Detach reason: the branch turned "Share this branch's books" off.



## bookclerk_library::control_plane::branch_catalog::DETACHED_WITHDRAWN

*Constant*: `&str`

Detach reason: a full pass of the branch's catalog no longer named it.



## bookclerk_library::control_plane::branch_catalog::FULL_CATALOG_PASS_EVERY

*Constant*: `chrono::Duration`

How long a branch's catalog sync resumes incrementally before the
library asks for a full pass again, so titles a branch no longer has
(removed, or from a sub-branch that left) are withdrawn within a day.



## bookclerk_library::control_plane::branch_catalog::accept_catalog_batch

*Function*

Records one batch of `node_id`'s shared titles under `fence`, then the
cursor after it. Every title is checked before any is written; a title
this library already holds for the branch's owner is skipped. With
`done` (the last batch of a pass) while a pass from the start is under
way, titles the pass did not name are withdrawn (detached); `done` on
a resumed incremental pass changes nothing more.

Every write runs in one transaction, each statement gated on the fence
still being live, the branch still sharing and its credential live: a
concurrent revoke, leave or stop-sharing either lands after the whole
batch (and detaches what it recorded) or the batch records nothing.

# Errors

Returns an error when a read or write fails.

```rust
fn accept_catalog_batch(store: &crate::store::LibraryStore, node_id: &str, fence: i64, cursor: &str, done: bool, titles: &[bookclerk_node_proto::CatalogTitle]) -> crate::error::Result<CatalogAccept>
```



## bookclerk_library::control_plane::branch_catalog::begin_catalog

*Function*

Opens (or reopens) `node_id`'s catalog sync with the settings the
branch reports: records them on its node row, bumps its catalog fence
(frames under an older fence are lost from now on), and answers where
to resume. A change of either setting restarts the pass from the start,
so titles the new setting excludes are withdrawn when the pass ends; so
does a session [`FULL_CATALOG_PASS_EVERY`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogfull_catalog_pass_every) after the last full pass.
With sharing off, every title the branch shared is marked detached
(`stopped_sharing`).

# Errors

Returns an error when a read or write fails.

```rust
fn begin_catalog(store: &crate::store::LibraryStore, node_id: &str, share_books: bool, include_branches: bool) -> crate::error::Result<CatalogBegin>
```



## bookclerk_library::control_plane::branch_catalog::branch_provenance

*Function*

The provenance text of titles shared by `node_id`.

```rust
fn branch_provenance(node_id: &str) -> String
```



## bookclerk_library::control_plane::branch_catalog::branch_title_counts

*Function*

[`BranchTitleCounts`](branch_catalog.md#bookclerk_librarycontrol_planebranch_catalogbranchtitlecounts) of `node_id`.

# Errors

Returns the read error.

```rust
fn branch_title_counts(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<BranchTitleCounts>
```



## bookclerk_library::control_plane::branch_catalog::catalog_fence_live

*Function*

True while `fence` is `node_id`'s live catalog fence and it may share.

# Errors

Returns an error when a read fails.

```rust
fn catalog_fence_live(store: &crate::store::LibraryStore, node_id: &str, fence: i64) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::branch_catalog::detach_branch_titles

*Function*

Marks every title `node_id` shares detached with `reason`; answers how
many changed. Files already delivered stay where they are.

# Errors

Returns the write error.

```rust
fn detach_branch_titles(store: &crate::store::LibraryStore, node_id: &str, reason: &str) -> crate::error::Result<u64>
```



## bookclerk_library::control_plane::branch_catalog::forwardable_branch_titles

*Function*

On a branch: the titles its own branches shared with it that it may
pass on (still shared, files delivered), oldest first.

# Errors

Returns the read error.

```rust
fn forwardable_branch_titles(store: &crate::store::LibraryStore) -> crate::error::Result<Vec<branch_titles::Model>>
```



## bookclerk_library::control_plane::branch_catalog::grant_catalog_upload

*Function*

Opens an upload of one shared title's files, bound to `node_id`'s
catalog `fence` and received by `receiving_node_id`.

Granted only while the fence is live, for a title the branch shares,
when every file is copied to each of `destinations` (this node's, the
primary first) under its manifest key, the files are exactly the
recorded manifest, no key is one another title here already uses
(`key_in_use`: the other title's audio, PDF or primary stem;
`key_of_another_book`: any of a library book's keys or their
sidecars), and no key lies in or below a folder holding another title's
audio (`folder_of_another_book`: a library book's, or another branch's
delivered title's). The keys are then claimed for this upload
in `title_files`, refused when another title holds one there.

# Errors

Returns an error when a read or the insert fails.

```rust
fn grant_catalog_upload(store: &crate::store::LibraryStore, node_id: &str, fence: i64, receiving_node_id: &str, store_id: &str, product_id: &str, files: &[bookclerk_node_proto::UploadFile], destinations: &[String]) -> crate::error::Result<super::uploads::UploadGrant>
```



## bookclerk_library::control_plane::branch_catalog::list_branch_titles

*Function*

Titles branches shared, of one branch or all, oldest first, at most
`limit`.

# Errors

Returns the read error.

```rust
fn list_branch_titles(store: &crate::store::LibraryStore, node_id: Option<&str>, limit: u64) -> crate::error::Result<Vec<BranchTitle>>
```



## bookclerk_library::control_plane::branch_catalog::pending_deliveries

*Function*

Up to `limit` of `node_id`'s shared titles whose files have not
arrived (or changed since).

# Errors

Returns the read error.

```rust
fn pending_deliveries(store: &crate::store::LibraryStore, node_id: &str, limit: usize) -> crate::error::Result<Vec<bookclerk_node_proto::CatalogRef>>
```



## bookclerk_library::control_plane::branch_catalog::remove_detached_branch_titles

*Function*

Removes detached entries ("from a branch that left" and the like), of
one branch or of every one; answers how many. Only the library entries
go: files already delivered stay in this library's outputs.

# Errors

Returns the write error.

```rust
fn remove_detached_branch_titles(store: &crate::store::LibraryStore, node_id: Option<&str>) -> crate::error::Result<u64>
```



