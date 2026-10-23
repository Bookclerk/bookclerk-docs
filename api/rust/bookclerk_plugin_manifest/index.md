# bookclerk_plugin_manifest

`plugin.toml` schema — parse, validate, and canonical format.

This crate is the Rust projection of the install-time plugin descriptor.
It is shared by the host (`bookclerk-plugin-host`), the workerd launcher
(`bookclerk-workerd`), and the Rust guest SDK author tools (`check` /
`fmt`). Language SDKs (TypeScript / Python) validate against the committed
JSON Schema in [`PLUGIN_TOML_SCHEMA_JSON`](bookclerk_plugin_manifest.md#bookclerk_plugin_manifestplugin_toml_schema_json) (`schema/plugin-toml.json`),
which must stay in sync with the types here.

# Audience

- **Host / launcher authors** — parse manifests at discovery, build
  [`EgressPolicy`](egress.md#bookclerk_plugin_manifestegressegresspolicy) for isolate injection, map network + OAuth bindings to
  jail policy via `PluginManifest::jail_network_need`.
- **Plugin authors / SDK tools** — validate and pretty-print `plugin.toml`
  without pulling host dependencies.

# Modules

| Module | Responsibility |
| --- | --- |
| [`egress`](egress.md) | Hostname allowlist policy, IDNA normalization, Pyodide hosts |
| [`error`](error.md) | Parse / validate error type |
| [`fmt`](fmt.md) | Canonical TOML serialization for `fmt --check` |
| [`id`](id.md) | Strict plugin id grammar |
| [`logo`](logo.md) | Remote URL vs embedded path logo validation |
| [`types`](types.md) | `PluginManifest` and nested capability / workerd tables |
| [`workerd_contract`](workerd_contract.md) | Compatibility date, Python flag allowlist, module extensions |

Product narrative: `docs/plugins.md` (network consent, egress matching,
packaging). ABI contract: `bookclerk-plugin-abi`.

# Examples

```
use bookclerk_plugin_manifest::parse;

let manifest = parse(r#"
api_version = 3
id = "echo"
runtime = "native"
command = "./echo"
entrypoints = ["cli"]

[capabilities.network]
mode = "deny"
"#).expect("valid plugin.toml");
assert_eq!(manifest.id, "echo");
```

## Modules

### [`bookclerk_plugin_manifest`](bookclerk_plugin_manifest.md)

*1 constant, 1 function, 8 modules*

### [`address`](address.md)

*1 struct, 5 functions*

### [`egress`](egress.md)

*2 constants, 2 structs, 6 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`fmt`](fmt.md)

*1 function*

### [`id`](id.md)

*1 function*

### [`logo`](logo.md)

*1 enum, 2 constants, 3 functions*

### [`types`](types.md)

*21 structs, 23 constants, 4 enums, 5 functions*

### [`workerd_contract`](workerd_contract.md)

*1 struct, 2 constants, 6 functions*

