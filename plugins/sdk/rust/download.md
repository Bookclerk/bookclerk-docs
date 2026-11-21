# bookclerk_plugin_sdk::download

Resumable downloads to a file for native guests.

Audience: storefront and other native guests that download large files
(an audiobook can be several gigabytes). [`Download::to_file`](#downloadto_file) streams
the body to `<dest>.part` and, when the connection drops, stalls or
the server answers a transient error, asks again for only the bytes it
is missing (`Range: bytes=<n>-`, with `If-Range` when the server named
a validator). The decisions about each answer are the sans-IO
functions in [`crate::resume`](resume.md); this module only does the IO.

The partial survives a failed call: a later call for the same `dest`
resumes it when the server gave a validator (kept beside it in
`<dest>.part.meta`), and starts over when it did not. On success the
partial is renamed onto `dest` and the metadata file is removed.

```rust
use bookclerk_plugin_sdk::download::Download;
let done = Download::new()
    .max_bytes(8 * 1024 * 1024 * 1024)
    .to_file(std::path::Path::new("/tmp/book.m4b"), || {
        http.get("https://cdn.example.com/book.m4b")
    })
    .await?;
println!("{} bytes, resumed {} times", done.bytes, done.resumes);
```

| Item | Kind | Summary |
| --- | --- | --- |
| [`Downloaded`](#downloaded) | struct | A finished download. |
| [`Download`](#download) | struct | Settings for one resumable download; see the module docs. |
| [`DownloadError`](#downloaderror) | enum | Why `Download::to_file` failed. |
| [`DEFAULT_IDLE_TIMEOUT`](#default_idle_timeout) | constant | Default longest wait for the next body chunk (or the response head) before the connection is treated as stalled and the range is asked for again. |
| [`DEFAULT_RETRIES`](#default_retries) | constant | Default number of attempts in a row that may fail without writing a byte before the download gives up. |
| [`DEFAULT_MAX_ATTEMPTS`](#default_max_attempts) | constant | Default cap on attempts in one call, progress or not, so a server that drops every connection after a few bytes cannot keep a call alive forever. |
| [`DEFAULT_BACKOFF`](#default_backoff) | constant | Default first wait between attempts; it doubles after each failed attempt without progress, up to `MAX_BACKOFF`. |
| [`MAX_BACKOFF`](#max_backoff) | constant | Longest wait between attempts, including a server's `Retry-After`. |

## Structs

### Downloaded

```rust
pub struct Downloaded {
    pub path: PathBuf,
    pub bytes: u64,
    pub resumes: u32,
}
```

A finished download.

**Fields**

- `path`: The file written (the `dest` passed in).
- `bytes`: Its length in bytes.
- `resumes`: How many times a dropped or refused transfer was picked up again
  from where it stopped (0 when the first attempt finished).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Download

```rust
pub struct Download {
    /* private fields */
}
```

Settings for one resumable download; see the [module docs](#).

#### Download::new

```rust
pub fn new() -> Self
```

Settings with the defaults: no size check or limit,
[`DEFAULT_IDLE_TIMEOUT`](#default_idle_timeout), [`DEFAULT_RETRIES`](#default_retries),
[`DEFAULT_MAX_ATTEMPTS`](#default_max_attempts) and [`DEFAULT_BACKOFF`](#default_backoff).

#### Download::expected_size

```rust
pub fn expected_size(self, bytes: u64) -> Self
```

Requires the file to be exactly `bytes` long (a store's stated
size). A server stating another length fails the download.

#### Download::max_bytes

```rust
pub fn max_bytes(self, bytes: u64) -> Self
```

Refuses a file longer than `bytes`, before reading it when the
server states its length and as soon as it passes the limit
otherwise.

#### Download::idle_timeout

```rust
pub fn idle_timeout(self, timeout: Duration) -> Self
```

Sets the longest wait for the response head or the next body
chunk before the attempt counts as stalled.

#### Download::retries

```rust
pub fn retries(self, retries: u32) -> Self
```

Sets how many attempts in a row may fail without writing a byte.

#### Download::max_attempts

```rust
pub fn max_attempts(self, attempts: u32) -> Self
```

Sets how many attempts one call may make in all.

#### Download::backoff

```rust
pub fn backoff(self, backoff: Duration) -> Self
```

Sets the first wait between failed attempts (doubling, capped at
[`MAX_BACKOFF`](#max_backoff)).

#### Download::to_file

```rust
pub async fn to_file<F>(
    &self,
    dest: &Path,
    request: F,
) -> Result<Downloaded, DownloadError>
where
    F: FnMut() -> RequestBuilder,
```

Downloads to `dest`, resuming across dropped connections.

`request` builds the `GET` for the file (URL, auth, cookies,
user agent); it is called once per attempt, and the download adds
`Range` and `If-Range` itself. Signed URLs that expire belong in
the caller's retry, not here: a `403` fails the call and keeps the
partial for the next one.

##### Arguments

* `dest` - The final path. `<dest>.part` and `<dest>.part.meta`
  are written beside it, so the directory must be writable and the
  path used by this download only.
* `request` - Builds a fresh request for each attempt.

##### Returns

The finished file's path, length and resume count.

##### Errors

Returns [`DownloadError`]: `Status` for a refusal, `Interrupted`
after the retries, `TooLarge` / `SizeMismatch` when the length is
wrong (the partial is removed), `Io` when the file cannot be
written.

Implements: `Clone`, `Debug`, `Default`

## Enums

### DownloadError

```rust
pub enum DownloadError {
    Status { status: StatusCode },
    Interrupted { written: u64, reason: String },
    TooLarge { limit: u64 },
    SizeMismatch { expected: u64, actual: u64 },
    Io(Error),
}
```

Why [`Download::to_file`](#downloadto_file) failed.

**Variants**

- `Status`: The server answered a status that is not a download (`403`,
  `404`, …), or kept answering a transient one past the retries.
  - `status`: The status the server answered last.
- `Interrupted`: The connection kept failing or stalling past the retries; the
  partial is kept, so a later call resumes it when it can.
  - `written`: Bytes the partial holds.
  - `reason`: The last transport failure, without the URL.
- `TooLarge`: The file is larger than [`Download::max_bytes`](#downloadmax_bytes) allows.
  - `limit`: The limit, in bytes.
- `SizeMismatch`: The server's length disagrees with [`Download::expected_size`](#downloadexpected_size), or
  the body ended at another length than the one the server stated.
  - `expected`: The length the caller or the server stated.
  - `actual`: The length received or stated instead.
- `Io`: Reading or writing the partial file failed.

#### DownloadError::is_transient

```rust
pub fn is_transient(&self) -> bool
```

True when calling again later may succeed: the connection failed,
or the server answered `408`, `429` or `5xx`.

Implements: `Debug`, `Display`, `Error`, `From<Error>`

## Constants

### DEFAULT_IDLE_TIMEOUT

```rust
pub const DEFAULT_IDLE_TIMEOUT: Duration = _;
```

Default longest wait for the next body chunk (or the response head)
before the connection is treated as stalled and the range is asked
for again.

### DEFAULT_RETRIES

```rust
pub const DEFAULT_RETRIES: u32 = 5;
```

Default number of attempts in a row that may fail without writing a
byte before the download gives up.

### DEFAULT_MAX_ATTEMPTS

```rust
pub const DEFAULT_MAX_ATTEMPTS: u32 = 64;
```

Default cap on attempts in one call, progress or not, so a server that
drops every connection after a few bytes cannot keep a call alive
forever.

### DEFAULT_BACKOFF

```rust
pub const DEFAULT_BACKOFF: Duration = _;
```

Default first wait between attempts; it doubles after each failed
attempt without progress, up to [`MAX_BACKOFF`](#max_backoff).

### MAX_BACKOFF

```rust
pub const MAX_BACKOFF: Duration = _;
```

Longest wait between attempts, including a server's `Retry-After`.
