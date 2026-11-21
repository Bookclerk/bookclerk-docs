# bookclerk_player_api::request

Sans-IO parsers for what player apps send the `/player` API.

Every parser takes bytes or text exactly as they arrived (a body, a
header value, a raw query string) and answers a typed, bounded value or
a [`RequestError`](#requesterror). The routes call these and nothing else on app
input, and the `player_request` fuzz target drives the same functions
against an independent oracle.

Apps send more fields than are read here (device details, cached copies
of objects the server sent); unknown fields are ignored, as an
Audiobookshelf server ignores them. A field that is read must have the
documented type and range, or the whole request is refused.

| Item | Kind | Summary |
| --- | --- | --- |
| [`LoginRequest`](#loginrequest) | struct | `POST /player/login`: a username and a player password. |
| [`DeviceInfo`](#deviceinfo) | struct | What a player reported about the device it plays on, kept for the session list and history. |
| [`PlayRequest`](#playrequest) | struct | `POST /player/api/items/{id}/play`: what the app can play. |
| [`SyncRequest`](#syncrequest) | struct | `POST /player/api/session/{id}/sync` and `.../close`: where the player is, in milliseconds. |
| [`LocalSession`](#localsession) | struct | A session a player kept while offline, sent later (`POST /player/api/session/local`, one entry of `.../local-all`). |
| [`ProgressPatch`](#progresspatch) | struct | A change to the caller's progress in one item (`PATCH /player/api/me/progress/{id}` and each batch entry). |
| [`BookmarkRequest`](#bookmarkrequest) | struct | A bookmark as an app sends it (`POST`/`PATCH /player/api/me/item/{id}/bookmark`). |
| [`ListQuery`](#listquery) | struct | The query of a list route (`/player/api/libraries/{id}/items` and the other paged lists). |
| [`DetailQuery`](#detailquery) | struct | The query of a single-entity or small-list route (`GET /api/items/{id}`, `/api/authors/{id}`, `/api/libraries/{id}`, `/search`, `/me/listening-sessions`). |
| [`RequestError`](#requesterror) | enum | Why a request was refused. |
| [`ListSort`](#listsort) | enum | How a list is ordered (`sort`). |
| [`ListFilter`](#listfilter) | enum | Which part of a library a list shows (`filter=<group>.<base64 value>`). |
| [`parse_login_request`](#parse_login_request) | function | Parses a sign-in body: a JSON object with string `username` and `password`. |
| [`parse_card_token`](#parse_card_token) | function | A card token as an app sends it (`Authorization: Bearer`, `?token=`, `x-refresh-token`, the socket's `auth` event). |
| [`parse_bearer`](#parse_bearer) | function | The token of an `Authorization: Bearer <token>` value. |
| [`parse_play_request`](#parse_play_request) | function | Parses a play request. An empty body counts as `{}`. |
| [`parse_sync_request`](#parse_sync_request) | function | Parses a sync or close body. An empty body counts as `{}` (a close with nothing to report). |
| [`parse_local_session`](#parse_local_session) | function | Parses `POST /player/api/session/local`: one offline session. |
| [`parse_local_sessions`](#parse_local_sessions) | function | Parses `POST /player/api/session/local-all`: `{"sessions": [...]}`. |
| [`parse_progress_patch`](#parse_progress_patch) | function | Parses a progress change body. |
| [`parse_progress_batch`](#parse_progress_batch) | function | Parses `PATCH /player/api/me/progress/batch/update`: an array of progress changes, each naming its item in `libraryItemId`. |
| [`parse_bookmark_request`](#parse_bookmark_request) | function | Parses a bookmark body: a number `time` in seconds and an optional string `title`. |
| [`parse_bookmark_time`](#parse_bookmark_time) | function | The bookmark a `DELETE /player/api/me/item/{id}/bookmark/{time}` path names, by its time in seconds. |
| [`parse_batch_get`](#parse_batch_get) | function | Parses `POST /player/api/items/batch/get`: `{"libraryItemIds": [...]}`. |
| [`parse_list_filter`](#parse_list_filter) | function | Parses a `filter` value (`group.base64`). |
| [`parse_list_query`](#parse_list_query) | function | Parses a list route's raw query string. |
| [`parse_detail_query`](#parse_detail_query) | function | Parses the query of a single-entity or small-list route. |
| [`LocalSessionEntry`](#localsessionentry) | type alias | One entry of a batch of local sessions: the session, or why it was refused together with the id the app gave it (when it gave one), so the answer can name the entry it refuses. |
| [`MAX_LOGIN_BODY_BYTES`](#max_login_body_bytes) | constant | Largest sign-in body (`POST /player/login`). |
| [`MAX_USERNAME_BYTES`](#max_username_bytes) | constant | Longest username a sign-in may carry, in bytes. |
| [`MAX_PASSWORD_BYTES`](#max_password_bytes) | constant | Longest password a sign-in may carry, in bytes. |
| [`MAX_BODY_BYTES`](#max_body_bytes) | constant | Largest body of a play, sync, progress or bookmark request. Apps send a device description with a play request, so this is larger than the fields read need. |
| [`MAX_BATCH_BODY_BYTES`](#max_batch_body_bytes) | constant | Largest body of a request that carries several items or sessions (`/api/me/progress/batch/update`, `/api/session/local-all`, `/api/items/batch/get`). An offline session as the apps send it holds a copy of the item, so each can be several kilobytes. |
| [`MAX_BATCH_ENTRIES`](#max_batch_entries) | constant | Most entries a batch request may carry. |
| [`MAX_BOOKMARK_TITLE_BYTES`](#max_bookmark_title_bytes) | constant | Longest bookmark title read, in bytes. |
| [`MAX_QUERY_BYTES`](#max_query_bytes) | constant | Longest raw query string read, in bytes. |
| [`MAX_SEARCH_BYTES`](#max_search_bytes) | constant | Longest search term read, in bytes. |
| [`MAX_PAGE_LIMIT`](#max_page_limit) | constant | Most items one list page may ask for. |
| [`CARD_TOKEN_LEN`](#card_token_len) | constant | Length of a card token: base64url without padding of 32 random bytes. |
| [`MAX_DEVICE_NAME_BYTES`](#max_device_name_bytes) | constant | Longest player name kept from a play request's device details. |

## Structs

### LoginRequest

```rust
pub struct LoginRequest {
    pub username: String,
    pub password: String,
}
```

`POST /player/login`: a username and a player password.

**Fields**

- `username`: The patron's login name or email, trimmed.
- `password`: The player password as typed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DeviceInfo

```rust
pub struct DeviceInfo {
    pub name: String,
}
```

What a player reported about the device it plays on, kept for the
session list and history.

**Fields**

- `name`: A name for the player, built from the app's client name and the
  device's model, at most [`MAX_DEVICE_NAME_BYTES`](#max_device_name_bytes) bytes.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### PlayRequest

```rust
pub struct PlayRequest {
    pub device: DeviceInfo,
    pub force_direct_play: bool,
    pub force_transcode: bool,
}
```

`POST /player/api/items/{id}/play`: what the app can play.

**Fields**

- `device`: The player's device details.
- `force_direct_play`: True when the app asked for the stored file even if it might not
  play it (`forceDirectPlay`).
- `force_transcode`: True when the app asked for a transcode (`forceTranscode`); the
  library has none, so this is answered as not playable.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### SyncRequest

```rust
pub struct SyncRequest {
    pub position_ms: Option<u64>,
    pub listened_ms: u64,
    pub duration_ms: Option<u64>,
}
```

`POST /player/api/session/{id}/sync` and `.../close`: where the player
is, in milliseconds.

**Fields**

- `position_ms`: The position (`currentTime`); `None` when the report leaves it out,
  which a close may.
- `listened_ms`: Time listened since the last report (`timeListened`).
- `duration_ms`: The recording's runtime as the player knows it (`duration`).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### LocalSession

```rust
pub struct LocalSession {
    pub id: String,
    pub item_id: String,
    pub position_ms: u64,
    pub listened_ms: u64,
    pub duration_ms: Option<u64>,
    pub updated_at_ms: Option<i64>,
}
```

A session a player kept while offline, sent later
(`POST /player/api/session/local`, one entry of `.../local-all`).

**Fields**

- `id`: The app's own id for the session, echoed in the answer.
- `item_id`: The item it played (`libraryItemId`).
- `position_ms`: Where it stopped.
- `listened_ms`: Time listened in it (`timeListening`).
- `duration_ms`: The recording's runtime as the player knew it.
- `updated_at_ms`: When it was last reported, epoch milliseconds (`updatedAt`).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ProgressPatch

```rust
pub struct ProgressPatch {
    pub position_ms: Option<u64>,
    pub duration_ms: Option<u64>,
    pub is_finished: Option<bool>,
    pub hide: Option<bool>,
}
```

A change to the caller's progress in one item
(`PATCH /player/api/me/progress/{id}` and each batch entry).

**Fields**

- `position_ms`: The new position (`currentTime`).
- `duration_ms`: The runtime the app knows (`duration`).
- `is_finished`: Finished or not (`isFinished`).
- `hide`: True to take the item off Continue listening
  (`hideFromContinueListening`).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### BookmarkRequest

```rust
pub struct BookmarkRequest {
    pub position_ms: u64,
    pub title: String,
}
```

A bookmark as an app sends it
(`POST`/`PATCH /player/api/me/item/{id}/bookmark`).

**Fields**

- `position_ms`: The marked place (`time`).
- `title`: The note (`title`), trimmed; may be empty.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ListQuery

```rust
pub struct ListQuery {
    pub limit: u64,
    pub page: u64,
    pub sort: ListSort,
    pub descending: bool,
    pub filter: Option<ListFilter>,
    pub collapse_series: bool,
    pub include_progress: bool,
    pub minified: bool,
    pub search: Option<String>,
}
```

The query of a list route (`/player/api/libraries/{id}/items` and the
other paged lists).

**Fields**

- `limit`: Rows per page; `0` asks for every row (bounded by
  [`MAX_PAGE_LIMIT`](#max_page_limit)).
- `page`: Page number, from 0.
- `sort`: The order.
- `descending`: True to reverse it (`desc=1`).
- `filter`: The filter, when one was given.
- `collapse_series`: True to fold works of one series into one row (`collapseseries=1`).
- `include_progress`: True when the caller's progress should ride along on each item
  (`include=progress`).
- `minified`: True when the minified item shape was asked for (`minified=1`).
- `search`: A search term (`q`), trimmed.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### DetailQuery

```rust
pub struct DetailQuery {
    pub expanded: bool,
    pub include_progress: bool,
    pub include_filterdata: bool,
    pub include_items: bool,
    pub include_series: bool,
    pub limit: Option<u64>,
    pub page: u64,
    pub search: Option<String>,
}
```

The query of a single-entity or small-list route (`GET
/api/items/{id}`, `/api/authors/{id}`, `/api/libraries/{id}`,
`/search`, `/me/listening-sessions`).

**Fields**

- `expanded`: `expanded=1`: the item with its tracks and files.
- `include_progress`: `include=progress`: the patron's progress rides along.
- `include_filterdata`: `include=filterdata`: a library with its filter values.
- `include_items`: `include=items`: an author with their items.
- `include_series`: `include=series`: an author with their series.
- `limit`: `limit` or `itemsPerPage`, at most [`MAX_PAGE_LIMIT`](#max_page_limit).
- `page`: `page`, from 0.
- `search`: `q`, trimmed; `None` when empty.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Enums

### RequestError

```rust
pub enum RequestError {
    TooLong(usize),
    NotJson,
    Shape(&'static str),
    OutOfRange(&'static str),
}
```

Why a request was refused.

**Variants**

- `TooLong`: The body is longer than its limit.
- `NotJson`: The body is not a JSON value of the expected kind.
- `Shape`: A field is missing or has the wrong type.
- `OutOfRange`: A field is outside its range.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### ListSort

```rust
pub enum ListSort {
    Title,
    Author,
    Added,
    Duration,
    Series,
}
```

How a list is ordered (`sort`).

**Variants**

- `Title`: By title (`media.metadata.title`, and anything not listed here).
- `Author`: By author (`media.metadata.authorName`, `...authorNameLF`).
- `Added`: By when the library got it (`addedAt`, `birthtimeMs`, `mtimeMs`).
- `Duration`: By runtime (`media.duration`, `size`).
- `Series`: By place in a series (`sequence`).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### ListFilter

```rust
pub enum ListFilter {
    Author(String),
    Series(String),
    NoSeries,
    Genre(String),
    Tag(String),
    Narrator(String),
    Publisher(String),
    Finished,
    NotFinished,
    InProgress,
    NotStarted,
    Unsupported,
}
```

Which part of a library a list shows (`filter=<group>.<base64 value>`).

**Variants**

- `Author`: Works by this author: an author id ([`crate::ids::facet_id`](ids.md#facet_id)).
- `Series`: Works in this series: a series id ([`crate::ids::facet_id`](ids.md#facet_id)).
- `NoSeries`: Works in no series (`series.no-series`).
- `Genre`: Works with this genre.
- `Tag`: Works the caller tagged with this tag.
- `Narrator`: Works with this narrator.
- `Publisher`: Works from this publisher.
- `Finished`: Works the caller finished.
- `NotFinished`: Works the caller has not finished.
- `InProgress`: Works the caller has started and not finished.
- `NotStarted`: Works the caller has not started.
- `Unsupported`: A filter group this library does not keep (languages, ebooks,
  missing metadata, feeds, ...): nothing matches.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_login_request

```rust
pub fn parse_login_request(body: &[u8]) -> Result<LoginRequest, RequestError>
```

Parses a sign-in body: a JSON object with string `username` and
`password`.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_LOGIN_BODY_BYTES`](#max_login_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an object,
[`RequestError::Shape`](#requesterror) when a field is missing or not a string, and
[`RequestError::OutOfRange`](#requesterror) for an empty or overlong field or a
control character.

### parse_card_token

```rust
pub fn parse_card_token(value: &str) -> Option<&str>
```

A card token as an app sends it (`Authorization: Bearer`, `?token=`,
`x-refresh-token`, the socket's `auth` event).

#### Arguments

* `value` - The token text, trimmed of surrounding spaces.

#### Returns

`None` unless it is exactly [`CARD_TOKEN_LEN`](#card_token_len) base64url characters,
the only shape this library mints; anything else is refused without a
lookup.

### parse_bearer

```rust
pub fn parse_bearer(value: &[u8]) -> Option<&str>
```

The token of an `Authorization: Bearer <token>` value.

#### Returns

`None` unless the scheme is `Bearer` (any case) and the token passes
[`parse_card_token`](#parse_card_token).

### parse_play_request

```rust
pub fn parse_play_request(body: &[u8]) -> Result<PlayRequest, RequestError>
```

Parses a play request. An empty body counts as `{}`.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BODY_BYTES`](#max_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an object, and
[`RequestError::Shape`](#requesterror) when a flag is not a boolean.

### parse_sync_request

```rust
pub fn parse_sync_request(body: &[u8]) -> Result<SyncRequest, RequestError>
```

Parses a sync or close body. An empty body counts as `{}` (a close with
nothing to report).

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BODY_BYTES`](#max_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an object,
[`RequestError::Shape`](#requesterror) for a field that is not a number, and
[`RequestError::OutOfRange`](#requesterror) for a negative or overlong length.

### parse_local_session

```rust
pub fn parse_local_session(body: &[u8]) -> Result<LocalSession, RequestError>
```

Parses `POST /player/api/session/local`: one offline session.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BATCH_BODY_BYTES`](#max_batch_body_bytes), and the field
errors of an offline session.

### parse_local_sessions

```rust
pub fn parse_local_sessions(body: &[u8]) -> Result<Vec<LocalSessionEntry>, RequestError>
```

Parses `POST /player/api/session/local-all`: `{"sessions": [...]}`.

#### Returns

One result per entry, in order: an entry that does not parse is its
own error, so one bad session does not refuse the rest.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BATCH_BODY_BYTES`](#max_batch_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an object,
[`RequestError::Shape`](#requesterror) when `sessions` is not an array, and
[`RequestError::OutOfRange`](#requesterror) past [`MAX_BATCH_ENTRIES`](#max_batch_entries) entries.

### parse_progress_patch

```rust
pub fn parse_progress_patch(body: &[u8]) -> Result<ProgressPatch, RequestError>
```

Parses a progress change body.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BODY_BYTES`](#max_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an object, and the field
errors.

### parse_progress_batch

```rust
pub fn parse_progress_batch(
    body: &[u8],
) -> Result<Vec<(String, ProgressPatch)>, RequestError>
```

Parses `PATCH /player/api/me/progress/batch/update`: an array of
progress changes, each naming its item in `libraryItemId`.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BATCH_BODY_BYTES`](#max_batch_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an array of objects,
[`RequestError::OutOfRange`](#requesterror) past [`MAX_BATCH_ENTRIES`](#max_batch_entries) entries, and
the field errors of any entry.

### parse_bookmark_request

```rust
pub fn parse_bookmark_request(body: &[u8]) -> Result<BookmarkRequest, RequestError>
```

Parses a bookmark body: a number `time` in seconds and an optional
string `title`.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BODY_BYTES`](#max_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an object,
[`RequestError::Shape`](#requesterror) when `time` is missing or a field has the wrong
type, and [`RequestError::OutOfRange`](#requesterror) for a length out of range or a
title over [`MAX_BOOKMARK_TITLE_BYTES`](#max_bookmark_title_bytes) bytes or with a control
character.

### parse_bookmark_time

```rust
pub fn parse_bookmark_time(time: &str) -> Option<u64>
```

The bookmark a `DELETE /player/api/me/item/{id}/bookmark/{time}` path
names, by its time in seconds.

#### Returns

`None` unless `time` is a plain non-negative decimal number in range.

### parse_batch_get

```rust
pub fn parse_batch_get(body: &[u8]) -> Result<Vec<String>, RequestError>
```

Parses `POST /player/api/items/batch/get`: `{"libraryItemIds": [...]}`.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_BODY_BYTES`](#max_body_bytes),
[`RequestError::NotJson`](#requesterror) for anything but an object,
[`RequestError::Shape`](#requesterror) when the list is missing or holds a non-string,
and [`RequestError::OutOfRange`](#requesterror) past [`MAX_BATCH_ENTRIES`](#max_batch_entries) ids or for
an id of the wrong shape.

### parse_list_filter

```rust
pub fn parse_list_filter(raw: &str) -> Option<ListFilter>
```

Parses a `filter` value (`group.base64`).

#### Returns

`None` for a value of the wrong shape; [`ListFilter::Unsupported`](#listfilter) for
a well-formed filter on a group this library does not keep.

### parse_list_query

```rust
pub fn parse_list_query(raw: &str) -> Result<ListQuery, RequestError>
```

Parses a list route's raw query string.

Unknown keys are ignored; a known key with a bad value refuses the
query.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_QUERY_BYTES`](#max_query_bytes), and
[`RequestError::OutOfRange`](#requesterror) naming the first bad value.

### parse_detail_query

```rust
pub fn parse_detail_query(raw: &str) -> Result<DetailQuery, RequestError>
```

Parses the query of a single-entity or small-list route.

Unknown keys are ignored; a known key with a bad value refuses the
query.

#### Errors

[`RequestError::TooLong`](#requesterror) past [`MAX_QUERY_BYTES`](#max_query_bytes), and
[`RequestError::OutOfRange`](#requesterror) naming the first bad value.

## Type aliases

### LocalSessionEntry

```rust
pub type LocalSessionEntry = Result<LocalSession, (Option<String>, RequestError)>;
```

One entry of a batch of local sessions: the session, or why it was
refused together with the id the app gave it (when it gave one), so the
answer can name the entry it refuses.

## Constants

### MAX_LOGIN_BODY_BYTES

```rust
pub const MAX_LOGIN_BODY_BYTES: usize = _;
```

Largest sign-in body (`POST /player/login`).

### MAX_USERNAME_BYTES

```rust
pub const MAX_USERNAME_BYTES: usize = 256;
```

Longest username a sign-in may carry, in bytes.

### MAX_PASSWORD_BYTES

```rust
pub const MAX_PASSWORD_BYTES: usize = 256;
```

Longest password a sign-in may carry, in bytes.

### MAX_BODY_BYTES

```rust
pub const MAX_BODY_BYTES: usize = _;
```

Largest body of a play, sync, progress or bookmark request. Apps send a
device description with a play request, so this is larger than the
fields read need.

### MAX_BATCH_BODY_BYTES

```rust
pub const MAX_BATCH_BODY_BYTES: usize = _;
```

Largest body of a request that carries several items or sessions
(`/api/me/progress/batch/update`, `/api/session/local-all`,
`/api/items/batch/get`). An offline session as the apps send it holds a
copy of the item, so each can be several kilobytes.

### MAX_BATCH_ENTRIES

```rust
pub const MAX_BATCH_ENTRIES: usize = 100;
```

Most entries a batch request may carry.

### MAX_BOOKMARK_TITLE_BYTES

```rust
pub const MAX_BOOKMARK_TITLE_BYTES: usize = 200;
```

Longest bookmark title read, in bytes.

### MAX_QUERY_BYTES

```rust
pub const MAX_QUERY_BYTES: usize = _;
```

Longest raw query string read, in bytes.

### MAX_SEARCH_BYTES

```rust
pub const MAX_SEARCH_BYTES: usize = 256;
```

Longest search term read, in bytes.

### MAX_PAGE_LIMIT

```rust
pub const MAX_PAGE_LIMIT: u64 = 1000;
```

Most items one list page may ask for.

### CARD_TOKEN_LEN

```rust
pub const CARD_TOKEN_LEN: usize = 43;
```

Length of a card token: base64url without padding of 32 random bytes.

### MAX_DEVICE_NAME_BYTES

```rust
pub const MAX_DEVICE_NAME_BYTES: usize = 64;
```

Longest player name kept from a play request's device details.
