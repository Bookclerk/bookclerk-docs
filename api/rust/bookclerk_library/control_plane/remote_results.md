**bookclerk_library > control_plane > remote_results**

# Module: control_plane::remote_results

## Contents

**Enums**

- [`ScanAccept`](#scanaccept) - Outcome of [`accept_scan_results`](remote_results.md#bookclerk_librarycontrol_planeremote_resultsaccept_scan_results).

**Functions**

- [`accept_scan_results`](#accept_scan_results) - Validates one batch of a Host's scan rows and writes it to the library.

**Constants**

- [`MAX_SCAN_RESULT_ROWS`](#max_scan_result_rows) - Most rows one results batch may carry.

---

## bookclerk_library::control_plane::remote_results::MAX_SCAN_RESULT_ROWS

*Constant*: `usize`

Most rows one results batch may carry.



## bookclerk_library::control_plane::remote_results::ScanAccept

*Enum*

Outcome of [`accept_scan_results`](remote_results.md#bookclerk_librarycontrol_planeremote_resultsaccept_scan_results).

**Variants:**
- `Accepted{ upserted: u64 }` - The batch was written.
- `Lost` - The fence no longer owns the job, or the node is no longer trusted
- `Refused(&'static str)` - The batch is not allowed; nothing was written (the reason is

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanAccept) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ScanAccept`



## bookclerk_library::control_plane::remote_results::accept_scan_results

*Function*

Validates one batch of a Host's scan rows and writes it to the library.

`books` are the rows as the guest produced them; their `uuid`,
`plugin_key` and `source` are ignored and set here (`source` from the
account row), so a Host cannot address an existing row by id or label
its rows as another plugin's.

# Errors

Returns an error when a read or a write fails.

```rust
fn accept_scan_results(store: &crate::store::LibraryStore, node_id: &str, job_id: &str, generation: i64, plugin_key: &str, books: Vec<crate::store::NewBook>) -> crate::error::Result<ScanAccept>
```



