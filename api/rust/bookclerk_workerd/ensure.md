**bookclerk_workerd > ensure**

# Module: ensure

## Contents

**Functions**

- [`ensure_workerd`](#ensure_workerd) - Ensure `dir/workerd` matches [`WORKERD_RELEASE_TAG`](pin.md#bookclerk_workerdpinworkerd_release_tag), downloading if needed.
- [`workerd_bin_path`](#workerd_bin_path) - Preferred install directory: beside this process, else `dir` argument from callers.

---

## bookclerk_workerd::ensure::ensure_workerd

*Function*

Ensure `dir/workerd` matches [`WORKERD_RELEASE_TAG`](pin.md#bookclerk_workerdpinworkerd_release_tag), downloading if needed.

Returns the path to the executable. Honors `BOOKCLERK_WORKERD_BIN` when set:
if that absolute path is a file and its sibling version stamp or `--version`
output matches the pin, it is returned. Otherwise ensure installs into `dir`
(the override path is not overwritten). A matching stamp without a binary is
a cache miss. `--version` uses fixed argv on the absolute path (no `PATH`
lookup). The selected cache directory is the trusted install root.

# Arguments

* `dir` - Directory path for this operation.

# Returns

On success, the inner `PathBuf` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn ensure_workerd(dir: &std::path::Path) -> anyhow::Result<std::path::PathBuf>
```



## bookclerk_workerd::ensure::workerd_bin_path

*Function*

Preferred install directory: beside this process, else `dir` argument from callers.

# Arguments

* `dir` - Directory path for this operation.

# Returns

`PathBuf` result.

```rust
fn workerd_bin_path(dir: &std::path::Path) -> std::path::PathBuf
```



