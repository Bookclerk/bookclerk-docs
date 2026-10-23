**bookclerk_plugin_catalog > extract**

# Module: extract

## Contents

**Functions**

- [`extract_archive`](#extract_archive) - Extract an archive into `dest`, refusing path traversal and oversized output.
- [`require_mutable_child`](#require_mutable_child) - Lexical child path under `root` safe for `remove_dir_all` / `rename` / create.
- [`require_under`](#require_under) - Returns `path` when it stays under `root` (canonical containment).
- [`safe_join`](#safe_join) - Join `dest` / `rel` while rejecting absolute paths and `..` escapes.
- [`sha256_bytes`](#sha256_bytes) - SHA-256 hex digest of bytes.
- [`sha256_file`](#sha256_file) - SHA-256 hex digest of a file.

**Constants**

- [`MAX_ARCHIVE_BYTES`](#max_archive_bytes) - Hard cap on archive file size before extraction.
- [`MAX_EXTRACTED_BYTES`](#max_extracted_bytes) - Hard cap on total extracted archive bytes.

---

## bookclerk_plugin_catalog::extract::MAX_ARCHIVE_BYTES

*Constant*: `u64`

Hard cap on archive file size before extraction.



## bookclerk_plugin_catalog::extract::MAX_EXTRACTED_BYTES

*Constant*: `u64`

Hard cap on total extracted archive bytes.



## bookclerk_plugin_catalog::extract::extract_archive

*Function*

Extract an archive into `dest`, refusing path traversal and oversized output.

# Errors

Returns an error when the operation fails.

```rust
fn extract_archive(archive: &std::path::Path, format: crate::target::ArchiveFormat, dest: &std::path::Path) -> crate::error::Result<()>
```



## bookclerk_plugin_catalog::extract::require_mutable_child

*Function*

Lexical child path under `root` safe for `remove_dir_all` / `rename` / create.

Unlike [`require_under`](extract.md#bookclerk_plugin_catalogextractrequire_under), this returns the intended directory entry (not a
symlink target) and refuses when the leaf exists as a symlink or when a
resolved existing path equals the trusted root (so cleanup cannot wipe the
root via an in-root alias). Missing leaves are validated via [`require_under`](extract.md#bookclerk_plugin_catalogextractrequire_under).

# Errors

Returns when the path escapes `root`, is a symlink leaf, or would mutate the root.

```rust
fn require_mutable_child(root: &std::path::Path, path: &std::path::Path) -> crate::error::Result<std::path::PathBuf>
```



## bookclerk_plugin_catalog::extract::require_under

*Function*

Returns `path` when it stays under `root` (canonical containment).

Use before filesystem sinks that already hold an absolute path built from a
trusted root (install dest, staging, receipt). Prefer [`safe_join`](extract.md#bookclerk_plugin_catalogextractsafe_join) when the
relative segment is still separate.

When `path` does not exist yet, canonicalizes the nearest existing ancestor
and rejoins the missing suffix (same approach as bookclerk-sandbox
`require_under_root`) so a symlinked parent such as
`root/.staging -> /outside` cannot pass a lexical `starts_with` check.

# Errors

Returns when `path` contains a `ParentDir` component, or escapes `root`
lexically or after canonicalize.

```rust
fn require_under(root: &std::path::Path, path: &std::path::Path) -> crate::error::Result<std::path::PathBuf>
```



## bookclerk_plugin_catalog::extract::safe_join

*Function*

Join `dest` / `rel` while rejecting absolute paths and `..` escapes.

# Errors

Returns an error when the operation fails.

```rust
fn safe_join(dest: &std::path::Path, rel: &std::path::Path) -> crate::error::Result<std::path::PathBuf>
```



## bookclerk_plugin_catalog::extract::sha256_bytes

*Function*

SHA-256 hex digest of bytes.

```rust
fn sha256_bytes(data: &[u8]) -> String
```



## bookclerk_plugin_catalog::extract::sha256_file

*Function*

SHA-256 hex digest of a file.

Callers must pass a path already contained under a trusted root (for
example via [`safe_join`](extract.md#bookclerk_plugin_catalogextractsafe_join) / [`require_under`](extract.md#bookclerk_plugin_catalogextractrequire_under)). Trusted roots may retain
intentional `..` spelling until the caller canonicalizes them; this hasher
does not re-litigate path spelling.

# Errors

Returns an error when the file cannot be read.

```rust
fn sha256_file(path: &std::path::Path) -> crate::error::Result<String>
```



