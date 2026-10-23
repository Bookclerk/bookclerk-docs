**bookclerk_plugin_catalog > payload**

# Module: payload

## Contents

**Functions**

- [`manifest_sha256`](#manifest_sha256) - SHA-256 of `plugin.toml` at `plugin_root`.
- [`payload_root_sha256`](#payload_root_sha256) - Deterministic payload root: sorted relative path + type + content hash.

**Constants**

- [`PAYLOAD_SKIP_NAMES`](#payload_skip_names) - Directory / file names excluded from the immutable payload (host-owned

---

## bookclerk_plugin_catalog::payload::PAYLOAD_SKIP_NAMES

*Constant*: `&[&str]`

Directory / file names excluded from the immutable payload (host-owned
receipts only). Packaged `data/` and `tmp/` are hashed: mutable plugin
state lives under `$FILES_DIR/plugin-state/<PluginKey fs-id>/`, not inside
the install tree.



## bookclerk_plugin_catalog::payload::manifest_sha256

*Function*

SHA-256 of `plugin.toml` at `plugin_root`.

# Errors

Returns an error when the file cannot be read.

```rust
fn manifest_sha256(plugin_root: &std::path::Path) -> crate::error::Result<String>
```



## bookclerk_plugin_catalog::payload::payload_root_sha256

*Function*

Deterministic payload root: sorted relative path + type + content hash.

Encoding (UTF-8, `\n`-terminated records, paths use `/`):

- file: `f\t{rel}\t{sha256}`
- directory: `d\t{rel}` (children listed separately)

Symlinks and path traversal (`..`) are rejected. The root directory itself
is not listed; an empty tree hashes the empty byte string.

# Errors

Returns an error on I/O failure, traversal, or a symlink in the payload.

```rust
fn payload_root_sha256(plugin_root: &std::path::Path) -> crate::error::Result<String>
```



