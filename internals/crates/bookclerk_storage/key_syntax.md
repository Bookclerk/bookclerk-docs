# bookclerk_storage::key_syntax

The host's key syntax, and how a storage backend turns a host key into
the path or object key it stores.

The host names what it stores with one syntax, whatever the destination:
segments joined by `/`, filled from the destination's naming template.
Each storage plugin translates a host key into the key it actually uses
(a file-system path for `local`, an object key for an object store) and
returns it; the host records the translated key. The host never decides a
destination's character set or length rules itself (storage destinations
ADR).

The syntax ([`HostKey`](#hostkey)):

- one or more segments joined by `/`, the last naming the file;
- no segment is empty, `.` or `..`;
- no segment holds `/`, `\` or a control character;
- at most [`MAX_HOST_KEY_BYTES`](#max_host_key_bytes) UTF-8 bytes in all.

The translators here are the ones the first-party plugins use:
[`translate_for_filesystem`](#translate_for_filesystem) and [`translate_for_object_store`](#translate_for_object_store).

| Item | Kind | Summary |
| --- | --- | --- |
| [`HostKey`](#hostkey) | struct | A key in the host's syntax: `/`-joined segments, validated on construction. |
| [`FilesystemRules`](#filesystemrules) | enum | Which file-system naming rules apply. |
| [`translate_for_filesystem`](#translate_for_filesystem) | function | Translates `key` into a relative path the file system under `rules` accepts, segment by segment. |
| [`translate_for_object_store`](#translate_for_object_store) | function | Translates `key` into an object key an S3-compatible store accepts under a prefix of `prefix_bytes` bytes. |
| [`MAX_HOST_KEY_BYTES`](#max_host_key_bytes) | constant | Largest host key, in UTF-8 bytes. Matches the S3 object-key limit, so a host key that fits fits an object store before its prefix. |
| [`MAX_FILESYSTEM_COMPONENT`](#max_filesystem_component) | constant | Largest path component most file systems accept, in UTF-8 bytes on Unix and UTF-16 units on Windows. |
| [`MAX_OBJECT_KEY_BYTES`](#max_object_key_bytes) | constant | Largest object key S3-compatible stores accept, in UTF-8 bytes, including the destination's prefix. |

## Structs

### HostKey

```rust
pub struct HostKey(/* private */);
```

A key in the host's syntax: `/`-joined segments, validated on
construction.

#### HostKey::parse

```rust
pub fn parse(key: &str) -> Result<Self>
```

Parses `key` as a host key.

##### Errors

Returns [`StorageError::InvalidKey`](README.md#storageerror) when `key` breaks the syntax in
the module docs.

#### HostKey::as_str

```rust
pub fn as_str(&self) -> &str
```

The key as text.

#### HostKey::segments

```rust
pub fn segments(&self) -> impl Iterator<Item = &str>
```

The key's segments in order; the last names the file.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

## Enums

### FilesystemRules

```rust
pub enum FilesystemRules {
    Unix,
    Windows,
}
```

Which file-system naming rules apply.

**Variants**

- `Unix`: Unix-like: only the component length matters.
- `Windows`: Windows: reserved characters and device names, no trailing dot or
  space, and the component length in UTF-16 units.

#### FilesystemRules::native

```rust
pub fn native() -> Self
```

The rules for the platform this process runs on.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### translate_for_filesystem

```rust
pub fn translate_for_filesystem(key: &HostKey, rules: FilesystemRules) -> String
```

Translates `key` into a relative path the file system under `rules`
accepts, segment by segment.

Each segment is made legal (on Windows: reserved characters replaced, a
device name prefixed with `_`, trailing dots and spaces dropped) and cut
to [`MAX_FILESYSTEM_COMPONENT`](#max_filesystem_component), keeping a short extension. The same
segment always translates the same way, so the files of one folder stay
in one folder.

### translate_for_object_store

```rust
pub fn translate_for_object_store(key: &HostKey, prefix_bytes: usize) -> String
```

Translates `key` into an object key an S3-compatible store accepts
under a prefix of `prefix_bytes` bytes.

Characters the S3 guidelines say to avoid are replaced, and when the
prefixed key would pass [`MAX_OBJECT_KEY_BYTES`](#max_object_key_bytes) the file name's stem is
shortened first, then the longest folder segments, so the key keeps its
folders and its extension.

## Constants

### MAX_HOST_KEY_BYTES

```rust
pub const MAX_HOST_KEY_BYTES: usize = 1024;
```

Largest host key, in UTF-8 bytes. Matches the S3 object-key limit, so a
host key that fits fits an object store before its prefix.

### MAX_FILESYSTEM_COMPONENT

```rust
pub const MAX_FILESYSTEM_COMPONENT: usize = 255;
```

Largest path component most file systems accept, in UTF-8 bytes on Unix
and UTF-16 units on Windows.

### MAX_OBJECT_KEY_BYTES

```rust
pub const MAX_OBJECT_KEY_BYTES: usize = 1024;
```

Largest object key S3-compatible stores accept, in UTF-8 bytes,
including the destination's prefix.
