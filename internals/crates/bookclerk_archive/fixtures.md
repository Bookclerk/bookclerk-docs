# bookclerk_archive::fixtures

Archive builders for tests and fuzz seeds, including archives a careful
tool would refuse to write.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ZipEntry`](#zipentry) | enum | How a fixture ZIP entry is stored. |
| [`TarEntry`](#tarentry) | enum | How a fixture tar entry is made. |
| [`zip_of`](#zip_of) | function | A ZIP archive of `entries`, names written exactly as given. |
| [`tar_of`](#tar_of) | function | A tar archive of `entries`. Names are written into the header bytes as given, so names a tar tool would refuse (`../x`, `/etc/x`) can be built. |
| [`gzip`](#gzip) | function | `data` inside one gzip stream. |

## Enums

### ZipEntry

```rust
pub enum ZipEntry<'a> {
    Stored(&'a str, &'a [u8]),
    Deflated(&'a str, &'a [u8]),
    Symlink(&'a str, &'a str),
    Dir(&'a str),
}
```

How a fixture ZIP entry is stored.

**Variants**

- `Stored`: A file stored without compression.
- `Deflated`: A file compressed with Deflate.
- `Symlink`: A symbolic link to a target.
- `Dir`: A folder.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### TarEntry

```rust
pub enum TarEntry<'a> {
    File(&'a str, &'a [u8]),
    Symlink(&'a str, &'a str),
    HardLink(&'a str, &'a str),
    Fifo(&'a str),
    Dir(&'a str),
}
```

How a fixture tar entry is made.

**Variants**

- `File`: A regular file.
- `Symlink`: A symbolic link to a target.
- `HardLink`: A hard link to an earlier entry.
- `Fifo`: A FIFO.
- `Dir`: A folder.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### zip_of

```rust
pub fn zip_of(entries: &[ZipEntry<'_>]) -> Vec<u8>
```

A ZIP archive of `entries`, names written exactly as given.

### tar_of

```rust
pub fn tar_of(entries: &[TarEntry<'_>]) -> Vec<u8>
```

A tar archive of `entries`. Names are written into the header bytes as
given, so names a tar tool would refuse (`../x`, `/etc/x`) can be built.

### gzip

```rust
pub fn gzip(data: &[u8]) -> Vec<u8>
```

`data` inside one gzip stream.
