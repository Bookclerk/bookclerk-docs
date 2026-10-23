**bookclerk_plugin_tools > package**

# Module: package

## Contents

**Functions**

- [`package_plugin`](#package_plugin) - Packages a plugin directory into `out_dir` as a `.tar.gz` plus `SHA256SUMS`.

---

## bookclerk_plugin_tools::package::package_plugin

*Function*

Packages a plugin directory into `out_dir` as a `.tar.gz` plus `SHA256SUMS`.

Native guests include the binary named by `command` (mode `0755` on Unix)
and name the archive with the host target triple. Workerd guests copy the
`modules_dir` tree and use a `-workerd` archive stem. Existing
`SHA256SUMS` lines for the same archive name are replaced.

# Arguments

* `plugin_dir` - Directory containing `plugin.toml` and guest payload.
* `out_dir` - Destination directory for the archive and checksums file
  (created if missing).

# Returns

Absolute or relative path to the written `.tar.gz` archive.

# Errors

Returns [`SdkError`](../bookclerk_plugin_sdk/error.md#bookclerk_plugin_sdkerrorsdkerror) when the manifest is invalid, required binaries /
modules are missing, or archive / checksum I/O fails.

```rust
fn package_plugin(plugin_dir: &std::path::Path, out_dir: &std::path::Path) -> bookclerk_plugin_sdk::Result<std::path::PathBuf>
```



