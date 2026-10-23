# bookclerk_plugin_manifest::fmt

Canonical `plugin.toml` formatting for SDK `fmt` / `fmt --check`.

Serializes a validated [`crate::PluginManifest`](types.md#pluginmanifest) with `toml::to_string_pretty`
and ensures a trailing newline so `fmt --check` diffs stay stable across tools.

## Functions

### format_manifest

```rust
pub fn format_manifest(manifest: &PluginManifest) -> Result<String>
```

Serializes a validated manifest to canonical TOML.

The output is suitable for rewriting `plugin.toml` on disk or comparing
against the on-disk file in `fmt --check`. Field order and pretty-print
style follow `toml` crate defaults; a trailing `\n` is always present.

#### Arguments

* `manifest` - Manifest that has already passed [`PluginManifest::validate`](types.md#pluginmanifestvalidate)
  (typically from [`PluginManifest::parse`](types.md#pluginmanifestparse)).

#### Returns

Pretty-printed TOML ending with a newline.

#### Errors

Returns [`crate::Error::TomlSer`](error.md#error) when serialization fails.

#### Examples

```rust
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
