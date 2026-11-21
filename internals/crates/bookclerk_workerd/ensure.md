# bookclerk_workerd::ensure

Download / refresh the pinned Cloudflare `workerd` binary.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ensure_workerd`](#ensure_workerd) | function | Ensure `dir/workerd` matches `WORKERD_RELEASE_TAG` and the pinned Pyodide bundle sits beside it, downloading either if needed. |
| [`pyodide_bundle_dir`](#pyodide_bundle_dir) | function | Directory beside `workerd_bin` that holds the pinned Pyodide bundle, which workerd reads through `--pyodide-bundle-disk-cache-dir`. |
| [`ensure_pyodide_bundle`](#ensure_pyodide_bundle) | function | Ensure `dir/``PYODIDE_BUNDLE_DIR``/``PYODIDE_BUNDLE_FILE` holds the pinned Pyodide bundle, downloading and checking it when missing or wrong. |
| [`workerd_bin_path`](#workerd_bin_path) | function | Preferred install directory: beside this process, else `dir` argument from callers. |

## Functions

### ensure_workerd

```rust
pub fn ensure_workerd(dir: &Path) -> Result<PathBuf>
```

Ensure `dir/workerd` matches [`WORKERD_RELEASE_TAG`](pin.md#workerd_release_tag) and the pinned Pyodide
bundle sits beside it, downloading either if needed.

Returns the path to the executable. The bundle lands in
[`PYODIDE_BUNDLE_DIR`](pin.md#pyodide_bundle_dir) beside the returned binary (see
[`ensure_pyodide_bundle`](#ensure_pyodide_bundle)). Honors `BOOKCLERK_WORKERD_BIN` when set:
if that absolute path is a file and its sibling version stamp or `--version`
output matches the pin, it is returned. Otherwise ensure installs into `dir`
(the override path is not overwritten). A matching stamp without a binary is
a cache miss. `--version` uses fixed argv on the absolute path (no `PATH`
lookup). The selected cache directory is the trusted install root.

#### Arguments

* `dir` - Directory path for this operation.

#### Returns

On success, the inner `PathBuf` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### pyodide_bundle_dir

```rust
pub fn pyodide_bundle_dir(workerd_bin: &Path) -> Option<PathBuf>
```

Directory beside `workerd_bin` that holds the pinned Pyodide bundle, which
workerd reads through `--pyodide-bundle-disk-cache-dir`.

#### Arguments

* `workerd_bin` - The `workerd` executable.

#### Returns

`<workerd_bin's directory>/`[`PYODIDE_BUNDLE_DIR`](pin.md#pyodide_bundle_dir), or `None` for a path
with no parent.

### ensure_pyodide_bundle

```rust
pub fn ensure_pyodide_bundle(dir: &Path) -> Result<PathBuf>
```

Ensure `dir/`[`PYODIDE_BUNDLE_DIR`](pin.md#pyodide_bundle_dir)`/`[`PYODIDE_BUNDLE_FILE`](pin.md#pyodide_bundle_file) holds the
pinned Pyodide bundle, downloading and checking it when missing or wrong.

Python isolates load this bundle at start. The jail gives workerd no
network of its own, so the bundle must already be on disk; workerd reads
the directory read-only.

#### Arguments

* `dir` - The directory that holds the `workerd` binary.

#### Returns

Path to the bundle file.

#### Errors

When the directory cannot be created, the download fails, or the bytes do
not match [`PYODIDE_BUNDLE_SHA256_HEX`](pin.md#pyodide_bundle_sha256_hex).

### workerd_bin_path

```rust
pub fn workerd_bin_path(dir: &Path) -> PathBuf
```

Preferred install directory: beside this process, else `dir` argument from callers.

#### Arguments

* `dir` - Directory path for this operation.

#### Returns

`PathBuf` result.
