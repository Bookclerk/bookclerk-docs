# bookclerk_workerd::ensure

Download / refresh the pinned Cloudflare `workerd` binary.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ensure_workerd`](#ensure_workerd) | function | Ensure `dir/workerd` matches `WORKERD_RELEASE_TAG`, downloading if needed. |
| [`workerd_bin_path`](#workerd_bin_path) | function | Preferred install directory: beside this process, else `dir` argument from callers. |

## Functions

### ensure_workerd

```rust
pub fn ensure_workerd(dir: &Path) -> Result<PathBuf>
```

Ensure `dir/workerd` matches [`WORKERD_RELEASE_TAG`](pin.md#workerd_release_tag), downloading if needed.

Returns the path to the executable. Honors `BOOKCLERK_WORKERD_BIN` when set:
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

### workerd_bin_path

```rust
pub fn workerd_bin_path(dir: &Path) -> PathBuf
```

Preferred install directory: beside this process, else `dir` argument from callers.

#### Arguments

* `dir` - Directory path for this operation.

#### Returns

`PathBuf` result.
