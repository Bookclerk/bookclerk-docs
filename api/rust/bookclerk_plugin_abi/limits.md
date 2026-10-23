**bookclerk_plugin_abi > limits**

# Module: limits

## Contents

**Structs**

- [`ScalarLimits`](#scalarlimits) - Negotiated numeric limits advertised at [`super::PluginDescribe`](rpc_types.md#bookclerk_plugin_abirpc_typesplugindescribe).

**Constants**

- [`MAX_EVENT_PAYLOAD_BYTES`](#max_event_payload_bytes) - Maximum decoded size of a domain-event scalar payload (not a stream).
- [`MAX_KV_KEY_BYTES`](#max_kv_key_bytes) - Longest `KvNamespace` key in UTF-8 bytes.
- [`MAX_KV_METADATA_BYTES`](#max_kv_metadata_bytes) - Largest `KvNamespace` metadata JSON text in bytes.
- [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes) - Largest `KvNamespace` value in bytes (a `[[kv_namespaces]]` row may lower
- [`MAX_LIST_PAGE`](#max_list_page) - Maximum objects in one `Destination.list` page.
- [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops) - Maximum already-separated operations in one plugin-owned migration.
- [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes) - Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one
- [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops) - Maximum already-separated operations across one `databaseMigrations`
- [`MAX_SCALAR_BYTES`](#max_scalar_bytes) - Maximum decoded size of an ordinary RPC scalar value (not a stream window).
- [`MAX_STREAM_WINDOW_BYTES`](#max_stream_window_bytes) - Maximum bytes returned by one `ByteSource.pull` (flow-control window).
- [`PRODUCT_API_VERSION`](#product_api_version) - Product ABI version (`apiVersion` / `plugin.toml` `api_version`).

---

## bookclerk_plugin_abi::limits::MAX_EVENT_PAYLOAD_BYTES

*Constant*: `u32`

Maximum decoded size of a domain-event scalar payload (not a stream).



## bookclerk_plugin_abi::limits::MAX_KV_KEY_BYTES

*Constant*: `u32`

Longest `KvNamespace` key in UTF-8 bytes.



## bookclerk_plugin_abi::limits::MAX_KV_METADATA_BYTES

*Constant*: `u32`

Largest `KvNamespace` metadata JSON text in bytes.



## bookclerk_plugin_abi::limits::MAX_KV_VALUE_BYTES

*Constant*: `u32`

Largest `KvNamespace` value in bytes (a `[[kv_namespaces]]` row may lower
its own cap but never raise it past this).



## bookclerk_plugin_abi::limits::MAX_LIST_PAGE

*Constant*: `u32`

Maximum objects in one `Destination.list` page.



## bookclerk_plugin_abi::limits::MAX_PLUGIN_MIGRATION_OPS

*Constant*: `u32`

Maximum already-separated operations in one plugin-owned migration.



## bookclerk_plugin_abi::limits::MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES

*Constant*: `u32`

Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one
`databaseMigrations` registration (Cap'n Proto framing is not counted).



## bookclerk_plugin_abi::limits::MAX_PLUGIN_MIGRATION_TOTAL_OPS

*Constant*: `u32`

Maximum already-separated operations across one `databaseMigrations`
registration (independent of the per-migration cap and of id+SQL UTF-8 bytes).



## bookclerk_plugin_abi::limits::MAX_SCALAR_BYTES

*Constant*: `u32`

Maximum decoded size of an ordinary RPC scalar value (not a stream window).



## bookclerk_plugin_abi::limits::MAX_STREAM_WINDOW_BYTES

*Constant*: `u32`

Maximum bytes returned by one `ByteSource.pull` (flow-control window).



## bookclerk_plugin_abi::limits::PRODUCT_API_VERSION

*Constant*: `u32`

Product ABI version (`apiVersion` / `plugin.toml` `api_version`).



## bookclerk_plugin_abi::limits::ScalarLimits

*Struct*

Negotiated numeric limits advertised at [`super::PluginDescribe`](rpc_types.md#bookclerk_plugin_abirpc_typesplugindescribe).

**Fields:**
- `max_scalar_bytes: u32` - [`MAX_SCALAR_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_scalar_bytes) (or a host-clamped lower value).
- `max_stream_window_bytes: u32` - [`MAX_STREAM_WINDOW_BYTES`](limits.md#bookclerk_plugin_abilimitsmax_stream_window_bytes) (or a host-clamped lower value).
- `max_list_page: u32` - [`MAX_LIST_PAGE`](limits.md#bookclerk_plugin_abilimitsmax_list_page) (or a host-clamped lower value).

**Methods:**

- `fn validate(self: Self) -> crate::Result<Self>` - Rejects zero or above-product-maxima limits.
- `fn intersect(self: Self, other: Self) -> Self` - Intersection of host-offered and guest-accepted limits (component-wise min).
- `fn clamp_list_limit(self: Self, limit: u32) -> u32` - Clamps a list page `limit` into `(0, max_list_page]`.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ScalarLimits`
- **From**
  - `fn from(value: ScalarLimitsDto) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScalarLimits) -> bool`



