**bookclerk_library > control_plane > uploads**

# Module: control_plane::uploads

## Contents

**Structs**

- [`TitleNames`](#titlenames) - The names a title's acquire writes on one destination, as the owner
- [`UploadDestination`](#uploaddestination) - One output destination of the receiving owner node an upload is
- [`UploadFinish`](#uploadfinish) - How the job's closing upload completes it.
- [`UploadGrantRequest`](#uploadgrantrequest) - What a Host asks to upload for one title.
- [`UploadSession`](#uploadsession) - One open upload, as the upload endpoint sees it.

**Enums**

- [`UploadCommit`](#uploadcommit) - Outcome of [`commit_upload`](uploads.md#bookclerk_librarycontrol_planeuploadscommit_upload).
- [`UploadGrant`](#uploadgrant) - Outcome of [`grant_upload`](uploads.md#bookclerk_librarycontrol_planeuploadsgrant_upload).

**Functions**

- [`abort_job_uploads`](#abort_job_uploads) - Aborts every open upload `node_id` was granted under the job's lease
- [`abort_upload`](#abort_upload) - Aborts an open upload (its fence was lost, its commit was refused for
- [`authorize_upload`](#authorize_upload) - The upload `token` opens: `upload_id`'s row when the token's hash
- [`commit_upload`](#commit_upload) - Commits an upload whose every file was received.
- [`extend_job_uploads`](#extend_job_uploads) - Extends the token of every open upload `node_id` holds under the job's
- [`grant_upload`](#grant_upload) - Opens an upload of one title of the acquire job `node_id` holds at
- [`job_titles`](#job_titles) - The owner's library rows the acquire `job` run by `node_id` covers: of
- [`load_upload`](#load_upload) - Loads one upload by id.
- [`names_book`](#names_book) - True when `book` is one of the titles `title` names.
- [`record_upload_file`](#record_upload_file) - Records that file `index` reached every destination and extends the
- [`upload_fence_live`](#upload_fence_live) - True while the upload's fence is the job's live lease for its node and
- [`upload_owns_key`](#upload_owns_key) - True while `key` on `destination` is still `session`'s own: its

**Constants**

- [`UPLOAD_NAME_WILDCARD`](#upload_name_wildcard) - In a `TitleNames::part_patterns` pattern: one or more characters of a
- [`UPLOAD_NUMBER_WILDCARD`](#upload_number_wildcard) - In a `TitleNames::part_patterns` pattern: one or more ASCII digits,
- [`UPLOAD_TOKEN_TTL_SECS`](#upload_token_ttl_secs) - How long a granted upload's token is accepted past its latest

---

## bookclerk_library::control_plane::uploads::TitleNames

*Struct*

The names a title's acquire writes on one destination, as the owner
plans them, all without their extension.

**Fields:**
- `audio_stems: Vec<String>` - The planned audio key of a single-file acquire (one per folder
- `part_patterns: Vec<String>` - Part and chapter keys, with [`UPLOAD_NAME_WILDCARD`](uploads.md#bookclerk_librarycontrol_planeuploadsupload_name_wildcard) and

**Methods:**

- `fn admits(self: &Self, key: &str) -> bool` - True when `key` is one of the title's audio keys with any extension
- `fn admits_audio(self: &Self, key: &str) -> bool` - True when `key` is one of the title's audio keys with any extension.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &TitleNames) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> TitleNames`
- **Clone**
  - `fn clone(self: &Self) -> TitleNames`



## bookclerk_library::control_plane::uploads::UPLOAD_NAME_WILDCARD

*Constant*: `char`

In a `TitleNames::part_patterns` pattern: one or more characters of a
file name (no `/`), where a part or chapter title goes. Storage keys
hold no control character, so the marker never clashes with one.



## bookclerk_library::control_plane::uploads::UPLOAD_NUMBER_WILDCARD

*Constant*: `char`

In a `TitleNames::part_patterns` pattern: one or more ASCII digits,
where a part or chapter number goes.



## bookclerk_library::control_plane::uploads::UPLOAD_TOKEN_TTL_SECS

*Constant*: `i64`

How long a granted upload's token is accepted past its latest
extension (the grant, an accepted file, or a renewal of the job's
lease).



## bookclerk_library::control_plane::uploads::UploadCommit

*Enum*

Outcome of [`commit_upload`](uploads.md#bookclerk_librarycontrol_planeuploadscommit_upload).

**Variants:**
- `Committed{ job_completed: bool }` - The title is committed (now or by an earlier identical commit).
- `Lost` - The fence was lost before the commit; nothing was written.
- `Refused(&'static str)` - Not allowed (the reason is stable).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &UploadCommit) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> UploadCommit`



## bookclerk_library::control_plane::uploads::UploadDestination

*Struct*

One output destination of the receiving owner node an upload is
granted for, with where the title may be written there.

The owner computes the scope itself, from its own library row of the
granted title and that destination's naming options, exactly as a local
acquire would name the title; the Host's keys are never trusted to say
where a title belongs. A folder only this title's files use admits any
key below it; a folder several titles share (or the destination's
root) admits only the title's own planned names ([`TitleNames`](uploads.md#bookclerk_librarycontrol_planeuploadstitlenames)), so a
Host trusted with one title cannot claim another title's file there.

**Fields:**
- `id: String` - The destination's id (`local`, `s3`).
- `title_folders: Vec<String>` - The folders the title's files may be written under on this
- `title_names: Option<TitleNames>` - The title's planned names, when its folder is shared with other

**Methods:**

- `fn admits(self: &Self, key: &str) -> bool` - True when `key` lies in this destination's scope for the title.
- `fn admits_primary(self: &Self, key: &str) -> bool` - True when `key` may be the title's primary audio key here: in scope

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &UploadDestination) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> UploadDestination`



## bookclerk_library::control_plane::uploads::UploadFinish

*Struct*

How the job's closing upload completes it.

**Generic Parameters:**
- 'a

**Fields:**
- `detail: &'a str` - Outcome text recorded as the job's progress.
- `download_episodes: bool` - Whether the receiving node acquires podcast episodes (its

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UploadFinish<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::uploads::UploadGrant

*Enum*

Outcome of [`grant_upload`](uploads.md#bookclerk_librarycontrol_planeuploadsgrant_upload).

**Variants:**
- `Granted{ upload_id: String, token: zeroize::Zeroizing<String>, expires_at: chrono::DateTime<chrono::Utc>, superseded: Vec<UploadSession> }` - The upload is open.
- `Lost` - The fence no longer owns the job, or the node is no longer trusted
- `Refused(&'static str)` - Not allowed (the reason is stable).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_library::control_plane::uploads::UploadGrantRequest

*Struct*

What a Host asks to upload for one title.

**Generic Parameters:**
- 'a

**Fields:**
- `book_uuid: &'a str` - Library id of the title. Empty for the job's closing upload, which
- `primary_key: &'a str` - Storage key of the title's primary audio file on the primary
- `pdf_key: Option<&'a str>` - Storage key of the companion PDF on the primary destination (one of
- `files: &'a [bookclerk_node_proto::UploadFile]` - Every file, in upload order.
- `destinations: &'a [UploadDestination]` - The receiving owner node's output destinations, the primary first,

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UploadGrantRequest<'a>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::uploads::UploadSession

*Struct*

One open upload, as the upload endpoint sees it.

**Fields:**
- `upload_id: String` - Upload id.
- `node_id: String` - Host node it was granted to.
- `job_id: String` - Acquire job.
- `generation: i64` - Lease generation it is bound to.
- `receiving_node_id: String` - Owner node that receives the files.
- `book_uuid: String` - Library id of the title (empty for the job's closing upload).
- `primary_key: String` - Storage key of the primary audio file.
- `pdf_key: Option<String>` - Storage key of the companion PDF, when uploaded.
- `destinations: Vec<String>` - The receiving node's destination ids at grant, the primary first;
- `files: Vec<bookclerk_node_proto::UploadFile>` - The declared files.
- `received: std::collections::BTreeSet<usize>` - Indexes of the files every destination has.
- `state: String` - `open`, `committed` or `aborted`.
- `catalog_digest: Option<String>` - For a branch's catalog upload (`Self::is_catalog`): the title's

**Methods:**

- `fn is_catalog(self: &Self) -> bool` - True for a branch's catalog upload, bound to the node's catalog
- `fn complete(self: &Self) -> bool` - True once every declared file was received.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> UploadSession`



## bookclerk_library::control_plane::uploads::abort_job_uploads

*Function*

Aborts every open upload `node_id` was granted under the job's lease
generation `generation` (the job finished or its fence was lost, so
none of them can commit); answers the aborted sessions.

# Errors

Returns an error when a read or write fails.

```rust
fn abort_job_uploads(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64) -> crate::error::Result<Vec<UploadSession>>
```



## bookclerk_library::control_plane::uploads::abort_upload

*Function*

Aborts an open upload (its fence was lost, its commit was refused for
a lost fence, or the destinations changed). Answers the session as it
stood when this call aborted it, its receipts final (no file is
recorded on an aborted upload), so the receiving node can delete every
copy it received; `None` when it was not open.

# Errors

Returns an error when a read or write fails.

```rust
fn abort_upload(store: &crate::store::LibraryStore, upload_id: &str) -> crate::error::Result<Option<UploadSession>>
```



## bookclerk_library::control_plane::uploads::authorize_upload

*Function*

The upload `token` opens: `upload_id`'s row when the token's hash
matches, the token has not expired, and the upload was neither aborted
nor (unless `allow_committed`) committed.

# Errors

Returns an error when the read fails.

```rust
fn authorize_upload(store: &crate::store::LibraryStore, upload_id: &str, token: &str, allow_committed: bool) -> crate::error::Result<Option<UploadSession>>
```



## bookclerk_library::control_plane::uploads::commit_upload

*Function*

Commits an upload whose every file was received.

A title's upload marks the title acquired under its primary key (and
its PDF, when uploaded) and publishes `book_acquired`; it never
completes the job, so `finish` must be `None`. The job's closing upload
(no title) must carry `finish`, and completes the job with its detail
as progress and gives its semaphore permits back, but only when no
title of the job is left unacquired and the job's cancel was not
requested; otherwise it is refused (`titles_remaining`,
`job_cancelled`) and nothing is written.

One canonical batch, every statement gated on the upload still being
open, the job still running under this upload's unexpired fence, the node's
credential still being live, the node still trusted with the title
(`commit_trust_gate`), and the upload still holding every key it was
granted (`upload_holds_its_keys`); trust and the keys are also
re-checked just before. A key another title or an acquire took since
the grant refuses the commit (`key_taken`) and records nothing. A repeat
of a commit that already landed answers `UploadCommit::Committed`
again without writing.

# Errors

Returns an error when a read or the batch fails.

```rust
fn commit_upload(store: &crate::store::LibraryStore, upload_id: &str, finish: Option<UploadFinish>, limits: super::job_routing::JobClaimLimits) -> crate::error::Result<UploadCommit>
```



## bookclerk_library::control_plane::uploads::extend_job_uploads

*Function*

Extends the token of every open upload `node_id` holds under the job's
lease generation `generation`, after that lease was renewed; answers
how many were extended.

# Errors

Returns an error when the write fails.

```rust
fn extend_job_uploads(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64) -> crate::error::Result<u64>
```



## bookclerk_library::control_plane::uploads::grant_upload

*Function*

Opens an upload of one title of the acquire job `node_id` holds at
`generation`, to be received by `receiving_node_id`.

Granted only when all of these hold: the fence is the job's live lease
for this node and the node is still trusted with the job; the job is an
acquire; the title exists, is one of the job's plugins and accounts (and
the job's title, when it names one), and for a Host trusted only with
its owner's accounts is one of that user's; and the manifest is well
formed, every key inside the title's own scope on its destination
([`UploadDestination`](uploads.md#bookclerk_librarycontrol_planeuploadsuploaddestination)) and none another title's: neither another
title's recorded audio, PDF or audio sidecar (`claims_another_books_key`)
nor any key recorded for another title on that destination
(`claim_upload_keys`, which then claims every key for this upload). An
earlier open upload of the same title by the same job is aborted: the
newest grant wins, and one overtaken by a newer grant of the title
while it was being made is refused (`superseded`) instead. A
grant refused or failed once claiming began claims nothing in the end:
the session row is inserted first and aborted again, and every key
claimed goes back to its prior holder (`undo_grant`).

# Errors

Returns an error when a read or the insert fails.

```rust
fn grant_upload(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64, receiving_node_id: &str, req: &UploadGrantRequest) -> crate::error::Result<UploadGrant>
```



## bookclerk_library::control_plane::uploads::job_titles

*Function*

The owner's library rows the acquire `job` run by `node_id` covers: of
the job's plugins ([`job_plugin_keys`](job_routing.md#bookclerk_librarycontrol_planejob_routingjob_plugin_keys)), of the accounts its account
selector names (`job_accounts`, so a job naming its account by label
or in another case covers that account's titles) and its title when it
names them, and, for a Host trusted only with its owner's accounts, one
of that user's. `None` when the node is trusted with nothing.

# Errors

Returns an error when a read fails.

```rust
fn job_titles(store: &crate::store::LibraryStore, node_id: &str, job: &crate::models::JobRecord) -> crate::error::Result<Option<Vec<crate::models::BookRecord>>>
```



## bookclerk_library::control_plane::uploads::load_upload

*Function*

Loads one upload by id.

# Errors

Returns an error when the read fails or the row is malformed.

```rust
fn load_upload(store: &crate::store::LibraryStore, upload_id: &str) -> crate::error::Result<Option<UploadSession>>
```



## bookclerk_library::control_plane::uploads::names_book

*Function*

True when `book` is one of the titles `title` names.

```rust
fn names_book(title: &str, book: &crate::models::BookRecord) -> bool
```



## bookclerk_library::control_plane::uploads::record_upload_file

*Function*

Records that file `index` reached every destination and extends the
upload's token. Answers false when the upload is no longer open (it was
aborted meanwhile: the caller deletes the copies it just wrote).

# Errors

Returns an error when a read or write fails.

```rust
fn record_upload_file(store: &crate::store::LibraryStore, upload_id: &str, index: usize) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::uploads::upload_fence_live

*Function*

True while the upload's fence is the job's live lease for its node and
the node is still trusted with the job.

# Errors

Returns an error when a read fails.

```rust
fn upload_fence_live(store: &crate::store::LibraryStore, session: &UploadSession) -> crate::error::Result<bool>
```



## bookclerk_library::control_plane::uploads::upload_owns_key

*Function*

True while `key` on `destination` is still `session`'s own: its
recorded latest writer is this upload. A later upload of the title, an
acquire run on this node, or another title may have written the key
since; the receiving node then leaves the file alone when it cleans up
after `session`, so a stale abort never deletes a replacement's file.

# Errors

Returns an error when the read fails.

```rust
fn upload_owns_key(store: &crate::store::LibraryStore, session: &UploadSession, destination: &str, key: &str) -> crate::error::Result<bool>
```



