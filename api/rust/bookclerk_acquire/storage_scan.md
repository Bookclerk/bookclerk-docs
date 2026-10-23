**bookclerk_acquire > storage_scan**

# Module: storage_scan

## Contents

**Structs**

- [`ScanCheckpoint`](#scancheckpoint) - Versioned scan progress stored in the job payload checkpoint.

**Functions**

- [`scan_storage`](#scan_storage) - Builds or resumes a durable identity/object index.

---

## bookclerk_acquire::storage_scan::ScanCheckpoint

*Struct*

Versioned scan progress stored in the job payload checkpoint.

**Fields:**
- `v: u32` - Schema version. Unknown versions fail closed.
- `op: String` - Always `storage_scan`.
- `instance_id: String` - `StorageBackend::instance_id` this cursor belongs to.
- `namespace: String` - List prefix (empty scans the namespace).
- `generation: String` - `storage_scan_rows.scan_id`.
- `phase: String` - `list` while paging, `apply` after the scan committed.
- `cursor: Option<String>` - Backend list cursor. `None` starts at the first page.
- `node_local: bool` - True when the list index or scratch is node-local.
- `placement: Option<String>` - Host placement for node-local bytes. Absent means the scan was recorded

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ScanCheckpoint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanCheckpoint) -> bool`



## bookclerk_acquire::storage_scan::scan_storage

*Function*

Builds or resumes a durable identity/object index.

# Errors

Returns [`AcquireError`](error.md#bookclerk_acquireerroracquireerror) when listing, the database, or a checkpoint write fails,
or when `prior` names a different storage instance.

```rust
fn scan_storage(library: &bookclerk_library::LibraryStore, storage: &dyn StorageBackend, fence: Option<&bookclerk_library::JobFence>, prior: Option<&bookclerk_plugin_abi::JobCheckpoint>, probe_metadata: bool) -> crate::error::Result<crate::reconcile::StorageIndex>
```



