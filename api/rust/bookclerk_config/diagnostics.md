**bookclerk_config > diagnostics**

# Module: diagnostics

## Contents

**Structs**

- [`BufferedEvent`](#bufferedevent) - Snapshot of a single redacted log event kept for crash / error uploads.
- [`DiagnosticsHandle`](#diagnosticshandle) - Process-global diagnostics handle installed by [`crate::logging::init_tracing_with`](logging.md#bookclerk_configlogginginit_tracing_with).
- [`UploadPayload`](#uploadpayload) - JSON body POSTed to the diagnostics collector `/submit` endpoint.

**Functions**

- [`global`](#global) - Accessor for the process-global diagnostics handle.

---

## bookclerk_config::diagnostics::BufferedEvent

*Struct*

Snapshot of a single redacted log event kept for crash / error uploads.

**Fields:**
- `ts_unix_ms: u64` - Event time as Unix milliseconds (wall clock).
- `level: String` - Tracing level name (`ERROR`, `WARN`, `INFO`, …).
- `target: String` - Tracing target / module path.
- `message: String` - Redacted primary message text.
- `fields: Vec<(String, String)>` - Additional redacted structured fields as `(name, value)` pairs.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> BufferedEvent`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::diagnostics::DiagnosticsHandle

*Struct*

Process-global diagnostics handle installed by [`crate::logging::init_tracing_with`](logging.md#bookclerk_configlogginginit_tracing_with).

**Methods:**

- `fn upload_enabled(self: &Self) -> bool` - Whether automatic upload is configured and ready (token/url present).
- `fn snapshot_events(self: &Self) -> Vec<BufferedEvent>` - Snapshot of the in-memory diagnostics ring (newest last).
- `fn version(self: &Self) -> &str` - Bookclerk version string recorded at subscriber install.
- `fn uploads_attempted(self: &Self) -> u64` - Number of upload attempts since init (successful or failed HTTP tries).
- `fn record_event(self: &Self, event: &Event)` - Record a redacted event into the ring buffer and maybe trigger an error-burst upload.
- `fn request_upload(self: &Self, trigger: &'static str)` - Request an upload of the current ring (e.g. after a failed daemon job).
- `fn upload_blocking(self: &Self, trigger: &str)` - Best-effort upload of the current ring buffer (blocking).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DiagnosticsHandle`



## bookclerk_config::diagnostics::UploadPayload

*Struct*

JSON body POSTed to the diagnostics collector `/submit` endpoint.

**Fields:**
- `trigger: String` - Why this upload was produced (`crash`, `error_burst`, `cli_error`, …).
- `version: String` - Bookclerk version string recorded at subscriber install.
- `os: String` - Host OS (`linux`, `macos`, `windows`, …).
- `arch: String` - CPU architecture (`x86_64`, `aarch64`, …).
- `distro: Option<String>` - Best-effort Linux distro id when detectable.
- `rustc_release: String` - `rustc` release baked in at build time (or `"unknown"`).
- `rustc_channel: String` - `rustc` channel baked in at build time (or `"unknown"`).
- `archived_at_unix_ms: u64` - Upload assembly time as Unix milliseconds.
- `events: Vec<BufferedEvent>` - Redacted ring-buffer snapshot (oldest first).

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> UploadPayload`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::diagnostics::global

*Function*

Accessor for the process-global diagnostics handle.

```rust
fn global() -> Option<&'static DiagnosticsHandle>
```



