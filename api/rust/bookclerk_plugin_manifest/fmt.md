**bookclerk_plugin_manifest > fmt**

# Module: fmt

## Contents

**Functions**

- [`format_manifest`](#format_manifest) - Serializes a validated manifest to canonical TOML.

---

## bookclerk_plugin_manifest::fmt::format_manifest

*Function*

Serializes a validated manifest to canonical TOML.

The output is suitable for rewriting `plugin.toml` on disk or comparing
against the on-disk file in `fmt --check`. Field order and pretty-print
style follow `toml` crate defaults; a trailing `\n` is always present.

# Arguments

* `manifest` - Manifest that has already passed `PluginManifest::validate`
  (typically from `PluginManifest::parse`).

# Returns

Pretty-printed TOML ending with a newline.

# Errors

Returns `crate::Error::TomlSer` when serialization fails.

# Examples

```
use bookclerk_plugin_manifest::{format_manifest, parse};

let m = parse(r#"
api_version = 3
id = "echo"
runtime = "native"
command = "./echo"
entrypoints = ["cli"]

[capabilities.network]
mode = "deny"
"#).unwrap();
let formatted = format_manifest(&m).unwrap();
assert!(formatted.ends_with('\n'));
assert_eq!(parse(&formatted).unwrap(), m);
```

```rust
fn format_manifest(manifest: &crate::types::PluginManifest) -> crate::error::Result<String>
```



