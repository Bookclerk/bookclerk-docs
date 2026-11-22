# bookclerk_storage::b2_api

Backblaze B2 native API replies, read into checked types.

The `b2` destination talks to B2 through its native API (version 3):
`b2_authorize_account`, upload URLs, large-file parts, file-name and
file-version listings, deletes and server-side copies. Every reply comes
from outside Bookclerk, so this module is the boundary that reads them:
pure functions over bytes, with no I/O, fuzzed by the `b2_reply` target.

The module also holds the request-side rules that must agree with what
it reads back: which file names B2 accepts ([`check_file_name`](#check_file_name)) and how
a name is percent-encoded into an upload header or a download URL
([`encode_file_name`](#encode_file_name)).

Replies are capped at [`MAX_B2_REPLY_BYTES`](#max_b2_reply_bytes); ids, tokens, URLs, names
and file-info values each have their own cap, so a hostile or broken
service cannot make the guest hold more than a bounded amount.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AuthToken`](#authtoken) | struct | An authorization token: for the account, or for one upload URL. |
| [`B2Url`](#b2url) | struct | An `https` URL B2 handed out (API, download or upload URL), with no user info, query or fragment. |
| [`FileId`](#fileid) | struct | A B2 file id: one version of one file name. |
| [`Bucket`](#bucket) | struct | A bucket an application key may reach. |
| [`Authorization`](#authorization) | struct | What `b2_authorize_account` granted. |
| [`UploadTarget`](#uploadtarget) | struct | Where one upload (a small file, or one part of a large file) goes. |
| [`FileVersion`](#fileversion) | struct | One version of one file, as B2 describes it. |
| [`NamesPage`](#namespage) | struct | One page of `b2_list_file_names`. |
| [`VersionsPage`](#versionspage) | struct | One page of `b2_list_file_versions`. |
| [`StoredPart`](#storedpart) | struct | What `b2_upload_part` stored. |
| [`DownloadHead`](#downloadhead) | struct | What a download (`GET` or `HEAD` on `/file/<bucket>/<name>`) says about the file in its response headers. |
| [`ApiError`](#apierror) | struct | An error reply from the API. |
| [`B2ReplyError`](#b2replyerror) | enum | Why a reply could not be read. |
| [`FileNameError`](#filenameerror) | enum | Why a file name is refused. |
| [`FileAction`](#fileaction) | enum | What a file version is. |
| [`check_file_name`](#check_file_name) | function | Checks a file name against B2's rules: 1 to `MAX_FILE_NAME_BYTES` bytes of UTF-8, no control characters or backslashes, and no leading, trailing or doubled `/`. |
| [`encode_file_name`](#encode_file_name) | function | Percent-encodes a file name (or a file-info value) the way B2 reads the `X-Bz-File-Name` header and the `/file/<bucket>/<name>` download path: every byte outside the unreserved set and `/` becomes `%XX` (uppercase). |
| [`decode_file_name`](#decode_file_name) | function | Decodes percent-encoding (`%XX`, and `+` as a space). |
| [`parse_authorization`](#parse_authorization) | function | Reads a `b2_authorize_account` reply. |
| [`parse_buckets`](#parse_buckets) | function | Reads a `b2_list_buckets` reply. |
| [`parse_upload_target`](#parse_upload_target) | function | Reads a `b2_get_upload_url` or `b2_get_upload_part_url` reply. |
| [`parse_file_version`](#parse_file_version) | function | Reads a reply that describes one file version: an upload, a finished large file, a copy, or `b2_start_large_file`. |
| [`parse_names_page`](#parse_names_page) | function | Reads a `b2_list_file_names` reply. |
| [`parse_versions_page`](#parse_versions_page) | function | Reads a `b2_list_file_versions` reply. |
| [`parse_part`](#parse_part) | function | Reads a `b2_upload_part` reply. |
| [`parse_download_headers`](#parse_download_headers) | function | Reads the headers of a download reply. |
| [`parse_api_error`](#parse_api_error) | function | Reads an error reply. Never fails: a body that is not B2's error JSON reads as code `unknown` with no message. |
| [`ReplyResult`](#replyresult) | type alias | Result alias for this module's parsers. |
| [`MAX_B2_REPLY_BYTES`](#max_b2_reply_bytes) | constant | Largest reply read, in bytes: a full listing of `MAX_LIST_FILES` versions, each with its name and file info, fits well inside. |
| [`MAX_LIST_FILES`](#max_list_files) | constant | Most files one listing reply may hold (B2's own `maxFileCount` limit). |
| [`MAX_FILE_NAME_BYTES`](#max_file_name_bytes) | constant | Longest file name B2 accepts, in UTF-8 bytes. |
| [`MAX_ID_BYTES`](#max_id_bytes) | constant | Longest id (account, bucket, file) accepted, in bytes. |
| [`MAX_TOKEN_BYTES`](#max_token_bytes) | constant | Longest authorization token accepted, in bytes. |
| [`MAX_URL_BYTES`](#max_url_bytes) | constant | Longest URL accepted, in bytes. |
| [`MAX_FILE_INFO_ENTRIES`](#max_file_info_entries) | constant | Most file-info entries one file may carry in a reply. |
| [`MAX_FILE_INFO_BYTES`](#max_file_info_bytes) | constant | Longest file-info key or value accepted, in bytes. |
| [`MAX_ERROR_TEXT_BYTES`](#max_error_text_bytes) | constant | Longest error code or message kept from an error reply, in bytes. |
| [`MAX_PARTS`](#max_parts) | constant | Most parts a large file may have (B2's limit). |
| [`MAX_DOWNLOAD_HEADERS`](#max_download_headers) | constant | Most response headers read from one download reply. |

## Structs

### AuthToken

```rust
pub struct AuthToken(/* private */);
```

An authorization token: for the account, or for one upload URL.

Its `Debug` output never shows the token.

#### AuthToken::parse

```rust
pub fn parse(token: &str) -> ReplyResult<Self>
```

Checks a token: 1 to [`MAX_TOKEN_BYTES`](#max_token_bytes) bytes of visible ASCII, so
it is safe in an `Authorization` header.

##### Errors

Returns [`B2ReplyError::Malformed`](#b2replyerror) for any other text.

#### AuthToken::expose

```rust
pub fn expose(&self) -> &str
```

The token, for the `Authorization` header.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### B2Url

```rust
pub struct B2Url(/* private */);
```

An `https` URL B2 handed out (API, download or upload URL), with no
user info, query or fragment.

#### B2Url::parse

```rust
pub fn parse(url: &str) -> ReplyResult<Self>
```

Checks a URL.

##### Errors

Returns [`B2ReplyError::Malformed`](#b2replyerror) when the URL is not `https`, is
over [`MAX_URL_BYTES`](#max_url_bytes), has no host, or holds user info, a query, a
fragment, whitespace or control characters.

#### B2Url::as_str

```rust
pub fn as_str(&self) -> &str
```

The URL, without a trailing `/`.

#### B2Url::host

```rust
pub fn host(&self) -> &str
```

The URL's host (with its port, when it has one).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### FileId

```rust
pub struct FileId(/* private */);
```

A B2 file id: one version of one file name.

#### FileId::parse

```rust
pub fn parse(id: &str) -> ReplyResult<Self>
```

Checks a file id: 1 to [`MAX_ID_BYTES`](#max_id_bytes) bytes of visible ASCII.

##### Errors

Returns [`B2ReplyError::Malformed`](#b2replyerror) for any other text.

#### FileId::as_str

```rust
pub fn as_str(&self) -> &str
```

The id as text.

Implements: `Clone`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### Bucket

```rust
pub struct Bucket {
    pub id: String,
    pub name: String,
}
```

A bucket an application key may reach.

**Fields**

- `id`: The bucket id, which API calls take.
- `name`: The bucket name, which download URLs take.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Authorization

```rust
pub struct Authorization {
    pub account_id: String,
    pub token: AuthToken,
    pub api_url: B2Url,
    pub download_url: B2Url,
    pub recommended_part_size: u64,
    pub absolute_minimum_part_size: u64,
    pub bucket: Option<Bucket>,
    pub name_prefix: Option<String>,
    pub capabilities: Vec<String>,
}
```

What `b2_authorize_account` granted.

**Fields**

- `account_id`: The account id.
- `token`: The token every other API call sends.
- `api_url`: Base URL of the API calls.
- `download_url`: Base URL of downloads.
- `recommended_part_size`: Part size B2 recommends for large files, in bytes.
- `absolute_minimum_part_size`: Smallest part (other than the last) B2 accepts, in bytes.
- `bucket`: The bucket the key is restricted to, when it is restricted to one.
- `name_prefix`: The file-name prefix the key is restricted to, when it is.
- `capabilities`: The capabilities the key holds (`listFiles`, `writeFiles`, ...).

#### Authorization::can

```rust
pub fn can(&self, capability: &str) -> bool
```

Whether the key holds `capability`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### UploadTarget

```rust
pub struct UploadTarget {
    pub url: B2Url,
    pub token: AuthToken,
}
```

Where one upload (a small file, or one part of a large file) goes.

**Fields**

- `url`: The URL to `POST` the bytes to.
- `token`: The token for that URL only.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### FileVersion

```rust
pub struct FileVersion {
    pub file_id: Option<FileId>,
    pub file_name: String,
    pub content_length: u64,
    pub content_sha1: Option<String>,
    pub content_type: Option<String>,
    pub upload_timestamp_ms: u64,
    pub action: FileAction,
    pub file_info: BTreeMap<String, String>,
}
```

One version of one file, as B2 describes it.

**Fields**

- `file_id`: The version's id.
- `file_name`: The full file name.
- `content_length`: The size in bytes (zero for hide markers and folders).
- `content_sha1`: Lowercase hex SHA-1 of the bytes, when B2 knows it (small files, or
  large files that recorded `large_file_sha1`).
- `content_type`: The stored content type.
- `upload_timestamp_ms`: When the version was uploaded, in UTC unix milliseconds.
- `action`: What the version is.
- `file_info`: File info (`X-Bz-Info-*` values) stored with the version.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NamesPage

```rust
pub struct NamesPage {
    pub files: Vec<FileVersion>,
    pub next_file_name: Option<String>,
}
```

One page of `b2_list_file_names`.

**Fields**

- `files`: The newest version of each name, in name order.
- `next_file_name`: Where the next page starts, when there is one.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### VersionsPage

```rust
pub struct VersionsPage {
    pub files: Vec<FileVersion>,
    pub next: Option<(String, FileId)>,
}
```

One page of `b2_list_file_versions`.

**Fields**

- `files`: Versions in name order, newest first within a name.
- `next`: Where the next page starts (name and file id), when there is one.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StoredPart

```rust
pub struct StoredPart {
    pub part_number: u32,
    pub content_sha1: String,
}
```

What `b2_upload_part` stored.

**Fields**

- `part_number`: The part's number (1 to [`MAX_PARTS`](#max_parts)).
- `content_sha1`: Lowercase hex SHA-1 of the part.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DownloadHead

```rust
pub struct DownloadHead {
    pub version: FileVersion,
    pub body_offset: u64,
    pub body_length: u64,
}
```

What a download (`GET` or `HEAD` on `/file/<bucket>/<name>`) says
about the file in its response headers.

**Fields**

- `version`: The version served. `content_length` is the whole file's size.
- `body_offset`: Offset in the file of this response's first body byte: the range
  start for a ranged read, 0 otherwise.
- `body_length`: Bytes in this response's body: the range length for a ranged read,
  the whole size otherwise.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ApiError

```rust
pub struct ApiError {
    pub status: u16,
    pub code: String,
    pub message: String,
}
```

An error reply from the API.

**Fields**

- `status`: The HTTP status.
- `code`: B2's error code (`expired_auth_token`, `not_found`, ...), or
  `unknown` when the body named none.
- `message`: B2's message, when there was one.

#### ApiError::expired_token

```rust
pub fn expired_token(&self) -> bool
```

Whether the account token expired, so a fresh authorization should
be tried once.

#### ApiError::retryable

```rust
pub fn retryable(&self) -> bool
```

Whether B2 asked the caller to try again later (busy, too many
requests, or an internal error).

#### ApiError::not_found

```rust
pub fn not_found(&self) -> bool
```

Whether the object (or file id) does not exist.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

## Enums

### B2ReplyError

```rust
pub enum B2ReplyError {
    TooLarge,
    Malformed(String),
}
```

Why a reply could not be read.

**Variants**

- `TooLarge`: The reply is over [`MAX_B2_REPLY_BYTES`](#max_b2_reply_bytes).
- `Malformed`: The reply is not the expected JSON shape, or a field breaks a rule.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### FileNameError

```rust
pub enum FileNameError {
    Empty,
    TooLong,
    Control,
    Slash,
    Backslash,
}
```

Why a file name is refused.

**Variants**

- `Empty`: The name is empty.
- `TooLong`: The name is over [`MAX_FILE_NAME_BYTES`](#max_file_name_bytes) bytes.
- `Control`: The name holds a control character (below U+0020, or U+007F).
- `Slash`: The name starts or ends with `/`, or holds `//`.
- `Backslash`: The name holds a backslash, which B2 refuses.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### FileAction

```rust
pub enum FileAction {
    Upload,
    Copy,
    Hide,
    Start,
    Folder,
}
```

What a file version is.

**Variants**

- `Upload`: A stored file.
- `Copy`: A file made by a server-side copy.
- `Hide`: A hide marker: the name reads as deleted.
- `Start`: A large file started and not yet finished.
- `Folder`: A virtual folder (only in listings with a delimiter).

#### FileAction::is_stored

```rust
pub fn is_stored(self) -> bool
```

Whether the version holds the file's bytes (an upload or a copy).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### check_file_name

```rust
pub fn check_file_name(name: &str) -> Result<(), FileNameError>
```

Checks a file name against B2's rules: 1 to [`MAX_FILE_NAME_BYTES`](#max_file_name_bytes)
bytes of UTF-8, no control characters or backslashes, and no leading,
trailing or doubled `/`.

#### Arguments

* `name` - the full file name, prefix included.

#### Errors

Returns the first rule the name breaks.

### encode_file_name

```rust
pub fn encode_file_name(name: &str) -> String
```

Percent-encodes a file name (or a file-info value) the way B2 reads the
`X-Bz-File-Name` header and the `/file/<bucket>/<name>` download path:
every byte outside the unreserved set and `/` becomes `%XX` (uppercase).

#### Arguments

* `name` - the text to encode.

#### Returns

The encoded text; [`decode_file_name`](#decode_file_name) gives `name` back.

### decode_file_name

```rust
pub fn decode_file_name(text: &str) -> Option<String>
```

Decodes percent-encoding (`%XX`, and `+` as a space).

#### Arguments

* `text` - encoded text, such as a file-info header value.

#### Returns

The decoded text, or `None` when an escape is broken or the bytes are
not UTF-8.

### parse_authorization

```rust
pub fn parse_authorization(bytes: &[u8]) -> ReplyResult<Authorization>
```

Reads a `b2_authorize_account` reply.

#### Arguments

* `bytes` - the reply body.

#### Returns

What the key may do and where to send calls.

#### Errors

Returns [`B2ReplyError`](#b2replyerror) for an oversized or malformed reply, a bad
URL, id or token, a part size of zero or a minimum above the
recommendation, or a restriction to a bucket that no longer exists.

### parse_buckets

```rust
pub fn parse_buckets(bytes: &[u8]) -> ReplyResult<Vec<Bucket>>
```

Reads a `b2_list_buckets` reply.

#### Arguments

* `bytes` - the reply body.

#### Returns

The buckets it lists (at most 1,000, B2's per-account limit).

#### Errors

Returns [`B2ReplyError`](#b2replyerror) for an oversized or malformed reply, or a bad
bucket id or name.

### parse_upload_target

```rust
pub fn parse_upload_target(bytes: &[u8]) -> ReplyResult<UploadTarget>
```

Reads a `b2_get_upload_url` or `b2_get_upload_part_url` reply.

#### Arguments

* `bytes` - the reply body.

#### Returns

The upload URL and its token.

#### Errors

Returns [`B2ReplyError`](#b2replyerror) for an oversized or malformed reply, or a bad
URL or token.

### parse_file_version

```rust
pub fn parse_file_version(bytes: &[u8]) -> ReplyResult<FileVersion>
```

Reads a reply that describes one file version: an upload, a finished
large file, a copy, or `b2_start_large_file`.

#### Arguments

* `bytes` - the reply body.

#### Returns

The version.

#### Errors

Returns [`B2ReplyError`](#b2replyerror) for an oversized or malformed reply, or a field
that breaks its rule.

### parse_names_page

```rust
pub fn parse_names_page(bytes: &[u8]) -> ReplyResult<NamesPage>
```

Reads a `b2_list_file_names` reply.

#### Arguments

* `bytes` - the reply body.

#### Returns

The page.

#### Errors

Returns [`B2ReplyError`](#b2replyerror) for an oversized or malformed reply, more than
[`MAX_LIST_FILES`](#max_list_files) files, a bad version, or a continuation name that
does not move past the page.

### parse_versions_page

```rust
pub fn parse_versions_page(bytes: &[u8]) -> ReplyResult<VersionsPage>
```

Reads a `b2_list_file_versions` reply.

#### Arguments

* `bytes` - the reply body.

#### Returns

The page.

#### Errors

Returns [`B2ReplyError`](#b2replyerror) for an oversized or malformed reply, more than
[`MAX_LIST_FILES`](#max_list_files) files, a bad version, or a continuation with a name
but no id (or the reverse).

### parse_part

```rust
pub fn parse_part(bytes: &[u8]) -> ReplyResult<StoredPart>
```

Reads a `b2_upload_part` reply.

#### Arguments

* `bytes` - the reply body.

#### Returns

The part's number and SHA-1.

#### Errors

Returns [`B2ReplyError`](#b2replyerror) for an oversized or malformed reply, a part
number outside 1 to [`MAX_PARTS`](#max_parts), or a SHA-1 that is not 40 hex
digits.

### parse_download_headers

```rust
pub fn parse_download_headers<'a, I>(headers: I) -> ReplyResult<DownloadHead>
where
    I: IntoIterator<Item = (&'a str, &'a [u8])>,
```

Reads the headers of a download reply.

#### Arguments

* `headers` - each header's name (any case) and raw value, as received.

#### Returns

The version served and the body's length.

#### Errors

Returns [`B2ReplyError`](#b2replyerror) when there are more than
[`MAX_DOWNLOAD_HEADERS`](#max_download_headers) headers, a required header (`Content-Length`,
`X-Bz-File-Id`, `X-Bz-File-Name`) is missing or repeated, a value is not
text, a number does not parse, `Content-Range` disagrees with
`Content-Length`, or file info is over its limits.

### parse_api_error

```rust
pub fn parse_api_error(status: u16, bytes: &[u8]) -> ApiError
```

Reads an error reply. Never fails: a body that is not B2's error JSON
reads as code `unknown` with no message.

#### Arguments

* `status` - the HTTP status.
* `bytes` - the reply body.

#### Returns

The error, with code and message cut to [`MAX_ERROR_TEXT_BYTES`](#max_error_text_bytes).

## Type aliases

### ReplyResult

```rust
pub type ReplyResult<T> = Result<T, B2ReplyError>;
```

Result alias for this module's parsers.

## Constants

### MAX_B2_REPLY_BYTES

```rust
pub const MAX_B2_REPLY_BYTES: usize = _;
```

Largest reply read, in bytes: a full listing of [`MAX_LIST_FILES`](#max_list_files)
versions, each with its name and file info, fits well inside.

### MAX_LIST_FILES

```rust
pub const MAX_LIST_FILES: usize = 10_000;
```

Most files one listing reply may hold (B2's own `maxFileCount` limit).

### MAX_FILE_NAME_BYTES

```rust
pub const MAX_FILE_NAME_BYTES: usize = 1024;
```

Longest file name B2 accepts, in UTF-8 bytes.

### MAX_ID_BYTES

```rust
pub const MAX_ID_BYTES: usize = 256;
```

Longest id (account, bucket, file) accepted, in bytes.

### MAX_TOKEN_BYTES

```rust
pub const MAX_TOKEN_BYTES: usize = 4096;
```

Longest authorization token accepted, in bytes.

### MAX_URL_BYTES

```rust
pub const MAX_URL_BYTES: usize = 2048;
```

Longest URL accepted, in bytes.

### MAX_FILE_INFO_ENTRIES

```rust
pub const MAX_FILE_INFO_ENTRIES: usize = 64;
```

Most file-info entries one file may carry in a reply.

### MAX_FILE_INFO_BYTES

```rust
pub const MAX_FILE_INFO_BYTES: usize = 8192;
```

Longest file-info key or value accepted, in bytes.

### MAX_ERROR_TEXT_BYTES

```rust
pub const MAX_ERROR_TEXT_BYTES: usize = 1024;
```

Longest error code or message kept from an error reply, in bytes.

### MAX_PARTS

```rust
pub const MAX_PARTS: u32 = 10_000;
```

Most parts a large file may have (B2's limit).

### MAX_DOWNLOAD_HEADERS

```rust
pub const MAX_DOWNLOAD_HEADERS: usize = 128;
```

Most response headers read from one download reply.
