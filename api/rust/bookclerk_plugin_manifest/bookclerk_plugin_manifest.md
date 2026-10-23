**bookclerk_plugin_manifest**

# Module: bookclerk_plugin_manifest

## Contents

**Modules**

- [`address`](#address) - Address-space policy independent of hostname matching.
- [`egress`](#egress) - Shared host/egress allowlist policy (workerd JS bridge + native mediator).
- [`error`](#error) - Manifest parse and semantic-validation errors.
- [`fmt`](#fmt) - Canonical `plugin.toml` formatting for SDK `fmt` / `fmt --check`.
- [`id`](#id) - Display-alias grammar shared by manifest parse, host registry, and SDKs.
- [`logo`](#logo) - Plugin logo validation: remote `http(s)` URL or relative image path.
- [`types`](#types) - `plugin.toml` types: manifest root and nested capability / workerd tables.
- [`workerd_contract`](#workerd_contract) - Author compatibility date, flag allowlist, and module-extension rules.

**Functions**

- [`parse`](#parse) - Parses and validates a `plugin.toml` document.

**Constants**

- [`PLUGIN_TOML_SCHEMA_JSON`](#plugin_toml_schema_json) - Embedded JSON Schema text for authoring tools (`check` in TS/Python SDKs).

---

## bookclerk_plugin_manifest::PLUGIN_TOML_SCHEMA_JSON

*Constant*: `&str`

Embedded JSON Schema text for authoring tools (`check` in TS/Python SDKs).

Identical on disk to `schema/plugin-toml.json`. Prefer this constant over
reading the file when packaging tools as a single crate artifact.



## Module: address

Address-space policy independent of hostname matching.

Default reachability is the public Internet. Loopback, RFC1918, link-local,
unique-local, metadata, and IPv4-mapped forms of those ranges stay denied
unless the operator grants an explicit CIDR. A public-redirect capability
never implies those ranges.



## Module: egress

Shared host/egress allowlist policy (workerd JS bridge + native mediator).

Semantics match `bookclerk-workerd` bridge `egress.js` and the product rules
in `docs/plugins.md`:

- **Initial** request hosts must match `EgressPolicy::domains` (`*.`
  prefix wildcards). Hosts and patterns are IDNA ToASCII–normalized;
  percent-encoded hosts fail closed.
- **Redirect** hops must stay on the fetch allowlist unless the operator
  grants `EgressPolicy::allow_undeclared_public_redirects`. That flag
  never implies loopback, RFC1918, link-local, or metadata.
- Hop count is capped by `EgressPolicy::max_redirects` (wire
  `maxRedirects`).
- **Python + outbound** workerd guests also receive
  [`PYODIDE_EGRESS_HOSTS`](egress.md#bookclerk_plugin_manifestegresspyodide_egress_hosts) in consent and egress lists.

Native-behind-workerd guests use the same policy through the inherited
CONNECT mux; ambient `AF_INET`/`AF_INET6` is denied by the sibling jail. See
[`crate::NetworkMode`](types.md#bookclerk_plugin_manifesttypesnetworkmode) and `crate::PluginManifest::validate`.



## Module: error

Manifest parse and semantic-validation errors.

All fallible entry points in this crate ([`crate::parse`](bookclerk_plugin_manifest.md#bookclerk_plugin_manifestparse),
[`crate::validate_plugin_id`](id.md#bookclerk_plugin_manifestidvalidate_plugin_id), [`crate::validate_logo`](logo.md#bookclerk_plugin_manifestlogovalidate_logo),
[`crate::format_manifest`](fmt.md#bookclerk_plugin_manifestfmtformat_manifest), and so on) return [`Result`](error.md#bookclerk_plugin_manifesterrorresult) with this
[`enum@Error`](error.md#bookclerk_plugin_manifesterrorerror) type. TOML serde failures are preserved via
`Error::TomlDe` / `Error::TomlSer`; semantic rules use
`Error::Message`.



## Module: fmt

Canonical `plugin.toml` formatting for SDK `fmt` / `fmt --check`.

Serializes a validated [`crate::PluginManifest`](types.md#bookclerk_plugin_manifesttypespluginmanifest) with `toml::to_string_pretty`
and ensures a trailing newline so `fmt --check` diffs stay stable across tools.



## Module: id

Display-alias grammar shared by manifest parse, host registry, and SDKs.

The `id` is a presentation alias unique within one host plugin namespace
(`$BOOKCLERK_FILES_DIR`). It is not a durable security identity — that is the
provenance-qualified PluginKey. The grammar is **non-lossy**:
characters that would need rewriting (for example `/` → `_`) are rejected
instead of sanitized, so `a/b` and `a_b` cannot collide as aliases.



## Module: logo

Plugin logo validation: remote `http(s)` URL or relative image path.

The optional `logo` key in `plugin.toml` is either a browser-loadable
remote URL (Settings / Accounts UI loads it directly) or a path under the
plugin install root that the host serves via
[`embedded_logo_api_path`](logo.md#bookclerk_plugin_manifestlogoembedded_logo_api_path). Absolute filesystem paths, parent segments,
and non-http(s) URL schemes are rejected.



## bookclerk_plugin_manifest::parse

*Function*

Parses and validates a `plugin.toml` document.

Equivalent to [`PluginManifest::parse`]: deserializes TOML then runs
`PluginManifest::validate`.

# Arguments

* `text` - Full contents of a `plugin.toml` file (UTF-8).

# Returns

A validated [`PluginManifest`](types.md#bookclerk_plugin_manifesttypespluginmanifest).

# Errors

Returns `Error::TomlDe` on malformed TOML, or `Error::Message` when
semantic validation fails (id grammar, runtime requirements, network
domains, logo, and so on).

# Examples

```
use bookclerk_plugin_manifest::parse;

let m = parse(r#"
api_version = 3
id = "sqlite"
runtime = "native"
command = "./bookclerk-plugin-database-sqlite"
entrypoints = ["databaseAdapter"]

[capabilities.network]
mode = "deny"
"#).unwrap();
assert_eq!(m.primary_family().as_str(), "database");
```

```rust
fn parse(text: &str) -> Result<PluginManifest>
```



## Module: types

`plugin.toml` types: manifest root and nested capability / workerd tables.

These structs are the serde projection of the install descriptor. Unknown
keys are rejected (`deny_unknown_fields`) so typos fail at parse time.
Semantic rules that cannot be expressed in serde alone live in
`PluginManifest::validate`. Product narrative: `docs/plugins.md`.



## Module: workerd_contract

Author compatibility date, flag allowlist, and module-extension rules.

These match the pinned local `workerd` binary. `crate::PluginManifest::validate`
and the TypeScript / Python author checkers enforce the same calendar, flag,
and module rules. A `compatibility_date` newer than the pin stays legal:
check and load warn, then run at the pin, because workerd refuses a date
past the one baked into the binary. `[[kv_namespaces]]` and `[queues]` are
host bindings served at `open`, so nothing here constrains them.



