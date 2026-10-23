**bookclerk_plugin_sdk > manifest_caps**

# Module: manifest_caps

## Contents

**Functions**

- [`manifest_capabilities`](#manifest_capabilities) - Parses `plugin.toml` source and returns the [`PluginCapabilities`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedplugincapabilities) a guest

---

## bookclerk_plugin_sdk::manifest_caps::manifest_capabilities

*Function*

Parses `plugin.toml` source and returns the [`PluginCapabilities`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedplugincapabilities) a guest
must advertise from `describe()`.

Keeping the manifest as the single source of truth means the host's
widening check (manifest vs describe vs operator grant) cannot drift: the
guest embeds the same file the host installed.

# Arguments

* `manifest_toml` - Contents of the plugin's `plugin.toml`, typically
  `include_str!("../plugin.toml")`.

# Returns

The typed capability declaration in manifest order.

# Errors

Returns `PluginErrorCode::Internal` when the manifest does not parse or
fails validation — a build defect, not a runtime condition.

# Examples

```
use bookclerk_plugin_sdk::manifest_capabilities;

let caps = manifest_capabilities(r#"
api_version = 3
id = "echo"
runtime = "native"
command = "./echo"
entrypoints = ["cli"]

[capabilities.network]
mode = "deny"
"#).unwrap();
assert_eq!(caps.entrypoints.len(), 1);
```

```rust
fn manifest_capabilities(manifest_toml: &str) -> Result<bookclerk_plugin_abi::PluginCapabilities, bookclerk_plugin_abi::PluginError>
```



