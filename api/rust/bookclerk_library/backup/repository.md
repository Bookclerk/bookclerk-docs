**bookclerk_library > backup > repository**

# Module: backup::repository

## Contents

**Structs**

- [`BackupRepository`](#backuprepository) - On-disk backup repository rooted at `files_dir/backups`.
- [`RepoLockGuard`](#repolockguard) - Held exclusive backup-repository lock (publication vs GC/prune).

---

## bookclerk_library::backup::repository::BackupRepository

*Struct*

On-disk backup repository rooted at `files_dir/backups`.

**Methods:**

- `fn open(files_dir: &Path) -> Result<Self>` - Opens (creating) the repository under `files_dir/backups`.
- `fn open_root(root: &Path) -> Result<Self>` - Opens `root` as a backups repository (`manifests/` + `objects/`).
- `fn root(self: &Self) -> &Path` - Repository root (`…/backups`).
- `fn lock_exclusive(self: &Self) -> Result<RepoLockGuard>` - Exclusive lock covering object publication and GC/prune.
- `fn put_object(self: &Self, object: &CanonicalObject) -> Result<String>` - Writes `object` if missing. Returns the uncompressed SHA-256 hex.
- `fn get_object(self: &Self, digest: &str) -> Result<CanonicalObject>` - Loads and verifies one object by uncompressed digest.
- `fn object_exists(self: &Self, digest: &str) -> bool` - True when an object file exists (does not imply the bytes are valid).
- `fn publish_manifest(self: &Self, manifest: &BackupManifest) -> Result<PathBuf>` - Publishes `manifest` only after all referenced objects exist **and**
- `fn read_manifest(self: &Self, id: &str) -> Result<BackupManifest>` - Reads and integrity-checks a published manifest.
- `fn list_manifests(self: &Self) -> Vec<BackupManifest>` - Lists published recovery-point manifests oldest-first. Staging files
- `fn list_manifests_strict(self: &Self) -> Result<Vec<BackupManifest>>` - Lists every published recovery-point manifest, failing if any cannot
- `fn delete_manifest(self: &Self, id: &str) -> Result<bool>` - Deletes one published manifest (not its objects). Returns whether a
- `fn gc_unreferenced_objects(self: &Self) -> Result<usize>` - Deletes objects not referenced by any retained published manifest.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BackupRepository`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::backup::repository::RepoLockGuard

*Struct*

Held exclusive backup-repository lock (publication vs GC/prune).

**Generic Parameters:**
- 'a



