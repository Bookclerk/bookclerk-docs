**bookclerk_plugin_abi > db**

# Module: db

## Contents

**Enums**

- [`DbConnectParams`](#dbconnectparams) - Host-private tagged connect params for first-party database guests.

**Functions**

- [`binding_values_from_adapter_config`](#binding_values_from_adapter_config) - Builds [`crate::BindingValues`](roles.md#bookclerk_plugin_abirolesbindingvalues) carrying the public author-facing
- [`binding_values_from_params`](#binding_values_from_params) - Builds [`crate::BindingValues`](roles.md#bookclerk_plugin_abirolesbindingvalues) from host-internal connect params.
- [`connect_params_from_bindings`](#connect_params_from_bindings) - Decodes host-internal connect params from `databaseAdapter` bindings.
- [`database_adapter_config_from_bindings`](#database_adapter_config_from_bindings) - Decodes the public [`crate::DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) from the bindings of

**Constants**

- [`DATABASE_CONTEXT_MEDIA_TYPE`](#database_context_media_type) - Media type for `crate::BindingValues::config` connect payloads.

---

## bookclerk_plugin_abi::db::DATABASE_CONTEXT_MEDIA_TYPE

*Constant*: `&str`

Media type for `crate::BindingValues::config` connect payloads.



## bookclerk_plugin_abi::db::DbConnectParams

*Enum*

Host-private tagged connect params for first-party database guests.

Travels only inside `crate::BindingValues::config` built by the
host ([`binding_values_from_params`](db.md#bookclerk_plugin_abidbbinding_values_from_params)); not part of the public plugin
author ABI. Discriminant is wire field `backend` with lowercase tags.
SQLite guests open `library.db` at `Self::Sqlite::sqlite_path` (also
injected as `BOOKCLERK_SQLITE_PATH`); D1 / Postgres receive host-injected
credentials in the params. Third-party adapters receive the public
[`crate::DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) payload instead.

**Variants:**
- `Sqlite{ plugin_data_dir: String, sqlite_path: Option<String>, binding: Option<String>, provision: bool }` - Local SQLite file backend (`backend: "sqlite"`).
- `D1{ plugin_data_dir: String, account_id: String, database_id: String, api_base: String, api_token: String, binding: Option<String>, database_name: Option<String>, provision: bool }` - Cloudflare D1 HTTP API backend (`backend: "d1"`).
- `Postgres{ plugin_data_dir: String, url: String, binding: Option<String>, database: Option<String>, provision: bool }` - PostgreSQL connection-string backend (`backend: "postgres"`).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DbConnectParams`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbConnectParams) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::db::binding_values_from_adapter_config

*Function*

Builds [`crate::BindingValues`](roles.md#bookclerk_plugin_abirolesbindingvalues) carrying the public author-facing
[`crate::DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) (granted settings + data dir) in the typed
`adapter` field; `config` stays empty so host-private connect params never
reach third-party adapters.

```rust
fn binding_values_from_adapter_config(config: &crate::DatabaseAdapterConfig) -> crate::BindingValues
```



## bookclerk_plugin_abi::db::binding_values_from_params

*Function*

Builds [`crate::BindingValues`](roles.md#bookclerk_plugin_abirolesbindingvalues) from host-internal connect params.

# Errors

Returns when JSON serialization fails.

```rust
fn binding_values_from_params(params: &DbConnectParams) -> crate::Result<crate::BindingValues>
```



## bookclerk_plugin_abi::db::connect_params_from_bindings

*Function*

Decodes host-internal connect params from `databaseAdapter` bindings.

# Errors

Returns when the bindings omit connect params or JSON is invalid.

```rust
fn connect_params_from_bindings(ctx: &crate::BindingValues) -> crate::Result<DbConnectParams>
```



## bookclerk_plugin_abi::db::database_adapter_config_from_bindings

*Function*

Decodes the public [`crate::DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) from the bindings of
a `databaseAdapter` open (third-party adapter bootstrap).

# Errors

Returns when the bindings carry host-private connect params instead of
an adapter bootstrap (empty `adapter.pluginDataDir`).

```rust
fn database_adapter_config_from_bindings(ctx: &crate::BindingValues) -> crate::Result<crate::DatabaseAdapterConfig>
```



