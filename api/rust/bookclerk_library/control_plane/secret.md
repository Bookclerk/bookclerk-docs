**bookclerk_library > control_plane > secret**

# Module: control_plane::secret

## Contents

**Structs**

- [`ClusterSecret`](#clustersecret) - Cluster id and fingerprint of the secret root aligned for this process.

**Functions**

- [`align_cluster_root`](#align_cluster_root) - Ensures this process's DEK is the cluster secret root.
- [`load_cluster_row`](#load_cluster_row) - Reads the singleton cluster row when present.

---

## bookclerk_library::control_plane::secret::ClusterSecret

*Struct*

Cluster id and fingerprint of the secret root aligned for this process.

The unwrapped DEK stays in the process cache (`configure_master_key_with`).
Callers that need the key use [`crate::require_master_key`](../master_key.md#bookclerk_librarymaster_keyrequire_master_key).

**Fields:**
- `cluster_id: String` - Cluster id stored beside the fingerprint.
- `secret_fingerprint: String` - SHA-256 hex of the DEK.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ClusterSecret`



## bookclerk_library::control_plane::secret::align_cluster_root

*Function*

Ensures this process's DEK is the cluster secret root.

* Existing fingerprint: the local key must match. A missing file is not minted.
* No fingerprint and sealed secrets present: the local key must unseal one
  of them, then the fingerprint is recorded. A missing or wrong key fails
  without writing a new root.
* No fingerprint and no sealed secrets: the local key (minted when absent)
  is recorded. Concurrent initializers: one fingerprint wins; a loser that
  minted a different key deletes that new file and fails.

On success the accepted DEK is installed in the process cache. The returned
value is only the cluster id and fingerprint.

# Errors

Returns an error when bootstrap material is missing or does not match the
database. Existing rows are not updated on failure.

```rust
fn align_cluster_root(store: &crate::store::LibraryStore, files_dir: &std::path::Path, password: Option<&str>) -> crate::error::Result<ClusterSecret>
```



## bookclerk_library::control_plane::secret::load_cluster_row

*Function*

Reads the singleton cluster row when present.

# Errors

Returns an error when the read fails.

```rust
fn load_cluster_row(store: &crate::store::LibraryStore) -> crate::error::Result<Option<cluster_identity::Model>>
```



