**bookclerk_plugin_integration_audiobookshelf > listening**

# Module: listening

## Contents

**Functions**

- [`collect_listening_snapshots`](#collect_listening_snapshots) - Collect listening-progress snapshots from ABS without writing the library DB.
- [`sync_listening_progress`](#sync_listening_progress) - Pull ABS user media progress into the library DB.

---

## bookclerk_plugin_integration_audiobookshelf::listening::collect_listening_snapshots

*Function*

Collect listening-progress snapshots from ABS without writing the library DB.

Used by both the in-process sync path and the external plugin guest RPC.

# Errors

Returns an error when the operation fails.

```rust
fn collect_listening_snapshots(client: &crate::client::AbsApiClient) -> bookclerk_integrations::Result<Vec<bookclerk_integrations::ListeningProgressSnapshot>>
```



## bookclerk_plugin_integration_audiobookshelf::listening::sync_listening_progress

*Function*

Pull ABS user media progress into the library DB.

Best-effort matches rows to `book_uuid` / `work_id` via ASIN, ISBN, or title.
Prefer calling this through `bookclerk_integrations::Integration::sync_listening_progress`
on the registered ABS adapter rather than from host binaries.

# Errors

Returns an error when the operation fails.

```rust
fn sync_listening_progress(library: &bookclerk_library::LibraryStore, client: &crate::client::AbsApiClient) -> bookclerk_integrations::Result<usize>
```



