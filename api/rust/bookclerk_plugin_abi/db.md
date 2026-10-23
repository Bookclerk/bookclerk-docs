# bookclerk_plugin_abi::db

Database-adapter binding payloads (host connect params + public adapter config).

Guests such as `sqlite` / `d1` / `postgres` implement the SeaORM proxy
boundary. The host never links SQL engines; it opens the library through
`PluginWorker.open` bindings + typed adapter sessions after Cap'n Proto
spawn.

The [`crate::BindingValues`](index.md#bindingvalues) handed to a `databaseAdapter` entrypoint carry
one of two bootstraps:

- `DbConnectParams` (feature `host`) — host-private serde type for
  first-party connect-param building; not public JSON RPC. Wire fields use
  camelCase; its `backend` tag is lowercase (`sqlite`, `d1`, `postgres`).
- [`crate::DatabaseAdapterConfig`](generated.md#databaseadapterconfig) (public) — generic bootstrap for
  third-party adapters: the operator's granted `[database.<id>]` table plus
  the scoped data dir. Travels typed in [`crate::BindingValues::adapter`](index.md#bindingvalues);
  read it with [`database_adapter_config_from_bindings`](#database_adapter_config_from_bindings).

Semantic capability limits live in [`crate::DbCapabilities`](db_execute.md#dbcapabilities)
(`crate::db_execute`); bootstrap metadata lives in [`crate::DbBootstrap`](db_execute.md#dbbootstrap).

| Item | Kind | Summary |
| --- | --- | --- |
| [`DbConnectParams`](#dbconnectparams) | enum | Host-private tagged connect params for first-party database guests. |
| [`binding_values_from_adapter_config`](#binding_values_from_adapter_config) | function | Builds `crate::BindingValues` carrying the public author-facing `crate::DatabaseAdapterConfig` (granted settings + data dir) in the typed `adapter` field; `config` stays empty so host-private connect params never reach third-party adapters. |
| [`database_adapter_config_from_bindings`](#database_adapter_config_from_bindings) | function | Decodes the public `crate::DatabaseAdapterConfig` from the bindings of a `databaseAdapter` open (third-party adapter bootstrap). |
| [`binding_values_from_params`](#binding_values_from_params) | function | Builds `crate::BindingValues` from host-internal connect params. |
| [`connect_params_from_bindings`](#connect_params_from_bindings) | function | Decodes host-internal connect params from `databaseAdapter` bindings. |
| [`DATABASE_CONTEXT_MEDIA_TYPE`](#database_context_media_type) | constant | Media type for `crate::BindingValues::config` connect payloads. |

## Enums

### DbConnectParams

```rust
pub enum DbConnectParams {
    Sqlite { plugin_data_dir: String, sqlite_path: Option<String>, binding: Option<String>, provision: bool },
    D1 { plugin_data_dir: String, account_id: String, database_id: String, api_base: String, api_token: String, binding: Option<String>, database_name: Option<String>, provision: bool },
    Postgres { plugin_data_dir: String, url: String, binding: Option<String>, database: Option<String>, provision: bool },
}
```

Host-private tagged connect params for first-party database guests.

Travels only inside [`crate::BindingValues::config`](index.md#bindingvaluesconfig) built by the
host ([`binding_values_from_params`](#binding_values_from_params)); not part of the public plugin
author ABI. Discriminant is wire field `backend` with lowercase tags.
SQLite guests open `library.db` at [`Self::Sqlite::sqlite_path`](#dbconnectparams) (also
injected as `BOOKCLERK_SQLITE_PATH`); D1 / Postgres receive host-injected
credentials in the params. Third-party adapters receive the public
[`crate::DatabaseAdapterConfig`](generated.md#databaseadapterconfig) payload instead.

**Variants**

- `Sqlite`: Local SQLite file backend (`backend: "sqlite"`).
  - `plugin_data_dir`: Scoped writable directory for this plugin
    (`…/plugin-state/<PluginKey fs-id>/data`, wire `pluginDataDir`).
  - `sqlite_path`: Absolute path to the DB file (wire `sqlitePath`). The sqlite jail
    grants this file and its journal sidecars at spawn.
  - `binding`: Named plugin database binding this open serves, if any. Binding
    opens use a dedicated connection at `sqlite_path` (the spawn env
    override never applies) so each binding is its own database file.
  - `provision`: When false, open an existing file and do not create a missing one.
- `D1`: Cloudflare D1 HTTP API backend (`backend: "d1"`).
  - `plugin_data_dir`: Scoped writable directory for this plugin (wire `pluginDataDir`).
  - `account_id`: Cloudflare account id for the D1 API (wire `accountId`).
  - `database_id`: D1 database UUID (wire `databaseId`). Empty for binding opens: the
    adapter resolves (and creates) the database by `databaseName`.
  - `api_base`: API base URL (for example `https://api.cloudflare.com/client/v4`).
  - `api_token`: Bearer / API token the host injects; guests must not read env for this.
  - `binding`: Named plugin database binding this open serves, if any.
  - `database_name`: D1 database name to resolve/provision for a binding open.
  - `provision`: When false, look up an existing D1 database and do not create it.
- `Postgres`: PostgreSQL connection-string backend (`backend: "postgres"`).
  - `plugin_data_dir`: Scoped writable directory for this plugin (wire `pluginDataDir`).
  - `url`: Full Postgres connection URL (host-injected; may contain secrets).
  - `binding`: Named plugin database binding this open serves, if any.
  - `database`: Isolated PostgreSQL database for a binding open (created if missing).
  - `provision`: When false, connect to an existing database and do not `CREATE DATABASE`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### binding_values_from_adapter_config

```rust
pub fn binding_values_from_adapter_config(config: &DatabaseAdapterConfig) -> BindingValues
```

Builds [`crate::BindingValues`](index.md#bindingvalues) carrying the public author-facing
[`crate::DatabaseAdapterConfig`](generated.md#databaseadapterconfig) (granted settings + data dir) in the typed
`adapter` field; `config` stays empty so host-private connect params never
reach third-party adapters.

### database_adapter_config_from_bindings

```rust
pub fn database_adapter_config_from_bindings(
    ctx: &BindingValues,
) -> Result<DatabaseAdapterConfig>
```

Decodes the public [`crate::DatabaseAdapterConfig`](generated.md#databaseadapterconfig) from the bindings of
a `databaseAdapter` open (third-party adapter bootstrap).

#### Errors

Returns when the bindings carry host-private connect params instead of
an adapter bootstrap (empty `adapter.pluginDataDir`).

### binding_values_from_params

```rust
pub fn binding_values_from_params(params: &DbConnectParams) -> Result<BindingValues>
```

Builds [`crate::BindingValues`](index.md#bindingvalues) from host-internal connect params.

#### Errors

Returns when JSON serialization fails.

### connect_params_from_bindings

```rust
pub fn connect_params_from_bindings(ctx: &BindingValues) -> Result<DbConnectParams>
```

Decodes host-internal connect params from `databaseAdapter` bindings.

#### Errors

Returns when the bindings omit connect params or JSON is invalid.

## Constants

### DATABASE_CONTEXT_MEDIA_TYPE

```rust
pub const DATABASE_CONTEXT_MEDIA_TYPE: &str = "application/vnd.bookclerk.db-connect+json";
```

Media type for [`crate::BindingValues::config`](index.md#bindingvaluesconfig) connect payloads.
