# bookclerk_storage::fs_kind

What kind of file system a local path lives on, so the local backend only
leans on hard links, inode numbers and atomic renames where the file
system keeps their promises.

Network file systems (NFS, SMB/CIFS, AFP, WebDAV, 9p, cluster file
systems) and FUSE mounts break several of them:

- A retried NFS `LINK` or `RENAME` can answer "already exists" or "not
  found" after the first attempt succeeded, so the answer no longer says
  whether the operation happened.
- SMB, AFP and many FUSE file systems have no hard links, or fake them.
- Inode numbers and status-change times can change after a remount or a
  cache eviction, so a signature built from them reports changes that
  never happened.

A library folder can hold another mount below it, so callers classify the
directory an operation touches, not the library root.

| Item | Kind | Summary |
| --- | --- | --- |
| [`FsKind`](#fskind) | enum | The kind of file system behind a path. |
| [`classify`](#classify) | function | Classifies the file system that holds `path`, which must exist. |
| [`classify_nearest`](#classify_nearest) | function | Classifies the nearest existing ancestor of `path` (itself included), so a target that does not exist yet is classified by the directory it will be created in. |
| [`device_of`](#device_of) | function | The device id of the file system holding `path`, where the platform has one. Two paths with different ids are on different file systems, so a rename or hard link between them cannot work. |

## Enums

### FsKind

```rust
pub enum FsKind {
    Local,
    Remote,
    Userspace,
    Unknown,
}
```

The kind of file system behind a path.

**Variants**

- `Local`: A disk the operating system manages directly (ext4, XFS, Btrfs, ZFS,
  APFS, NTFS, FAT and the like).
- `Remote`: A network or cluster file system, or a Windows network drive.
- `Userspace`: A FUSE (or macFUSE) mount: its guarantees depend on the program
  behind it, so it is treated like a remote file system.
- `Unknown`: The kind could not be determined; treated like a remote file system.

#### FsKind::is_local

```rust
pub fn is_local(self) -> bool
```

True when hard links, `rename` and inode identity behave as on a
local disk, so a hard link can publish a file atomically and an
inode number can be part of a change signature.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### classify

```rust
pub fn classify(path: &Path) -> FsKind
```

Classifies the file system that holds `path`, which must exist.

Returns [`FsKind::Unknown`](#fskind) when the platform gives no answer.

### classify_nearest

```rust
pub fn classify_nearest(path: &Path) -> FsKind
```

Classifies the nearest existing ancestor of `path` (itself included), so
a target that does not exist yet is classified by the directory it will
be created in.

### device_of

```rust
pub fn device_of(path: &Path) -> Option<u64>
```

The device id of the file system holding `path`, where the platform has
one. Two paths with different ids are on different file systems, so a
rename or hard link between them cannot work.
