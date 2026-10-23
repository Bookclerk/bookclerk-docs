**bookclerk_plugin_abi**

# Module: bookclerk_plugin_abi

## Contents

**Modules**

- [`backup_ops`](#backup_ops) - Host-facing backup/restore primitives (Send + Sync).
- [`db`](#db) - Database-adapter binding payloads (host connect params + public adapter config).
- [`db_execute`](#db_execute) - Typed Cap'n database data-plane (`ExecuteRequest` / `ExecuteReply`) and
- [`db_value`](#db_value) - Universal Cap'n database value domain (`DbValue`).
- [`error`](#error) - Plugin error model shared by host and guests.
- [`generated`](#generated) - GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp.
- [`guest_sql`](#guest_sql) - Host-side grammar and scope checks for guest-authored typed SQL.
- [`json_bytes`](#json_bytes) - Serde helpers projecting Cap'n Proto `Data` fields as base64 text in JSON.
- [`kind`](#kind) - Output-plugin configuration DTOs carried as `application/json` inside
- [`methods`](#methods) - Role capability method names (camelCase on the wire).
- [`package_version`](#package_version) - `plugin.toml` `version` as a semantic version.
- [`plugin_capnp`](#plugin_capnp) - Generated Cap'n Proto RPC interfaces (`schema/plugin.capnp`).
- [`plugin_host_capnp`](#plugin_host_capnp) - Host-private Cap'n Proto RPC interfaces (`schema/plugin_host.capnp`).
- [`settings_schema`](#settings_schema) - Operator-facing plugin settings schema (`[[settings]]` in `plugin.toml`).
- [`sql_desugar`](#sql_desugar) - Host semantic desugars for canonical Bookclerk SQL.
- [`sql_types`](#sql_types) - Bookclerk SQL v1 types, CAST matrix, and fail-closed expression checking.

**Constants**

- [`PLUGIN_TOML_SCHEMA_JSON`](#plugin_toml_schema_json) - Embedded JSON Schema for install `plugin.toml` files (shared with language

---

## bookclerk_plugin_abi::PLUGIN_TOML_SCHEMA_JSON

*Constant*: `&str`

Embedded JSON Schema for install `plugin.toml` files (shared with language
SDK author tools and `bookclerk plugins` validation).



## Module: backup_ops

Host-facing backup/restore primitives (Send + Sync).

[`crate::AdapterDatabaseSession`](roles.md#bookclerk_plugin_abirolesadapterdatabasesession) methods are `?Send` Cap'n stubs. Library
backup orchestration stores a [`AdapterBackupOps`](backup_ops.md#bookclerk_plugin_abibackup_opsadapterbackupops) handle so capture/restore
can call identity and catalog primitives without emitting dialect SQL.



## Module: db

Database-adapter binding payloads (host connect params + public adapter config).

Guests such as `sqlite` / `d1` / `postgres` implement the SeaORM proxy
boundary. The host never links SQL engines; it opens the library through
`PluginWorker.open` bindings + typed adapter sessions after Cap'n Proto
spawn.

The [`crate::BindingValues`](roles.md#bookclerk_plugin_abirolesbindingvalues) handed to a `databaseAdapter` entrypoint carry
one of two bootstraps:

- `DbConnectParams` (feature `host`) — host-private serde type for
  first-party connect-param building; not public JSON RPC. Wire fields use
  camelCase; its `backend` tag is lowercase (`sqlite`, `d1`, `postgres`).
- [`crate::DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) (public) — generic bootstrap for
  third-party adapters: the operator's granted `[database.<id>]` table plus
  the scoped data dir. Travels typed in `crate::BindingValues::adapter`;
  read it with [`database_adapter_config_from_bindings`](db.md#bookclerk_plugin_abidbdatabase_adapter_config_from_bindings).

Semantic capability limits live in [`crate::DbCapabilities`](db_execute.md#bookclerk_plugin_abidb_executedbcapabilities)
(`crate::db_execute`); bootstrap metadata lives in [`crate::DbBootstrap`](db_execute.md#bookclerk_plugin_abidb_executedbbootstrap).



## Module: db_execute

Typed Cap'n database data-plane (`ExecuteRequest` / `ExecuteReply`) and
control-plane (`DbCapabilities`) mirrors of `plugin.capnp`.

Hosts call `AdapterDatabaseSession.capabilities` and
`AdapterDatabaseSession.execute`. Granted job SQL uses `GuestDatabase.execute`.



## Module: db_value

Universal Cap'n database value domain (`DbValue`).

Baseline cells are typed null, bool, int64, finite float64, UTF-8 text, and
bytes. SeaORM and JSON bridges live in `bookclerk-db-exec` at the adapter edge.



## Module: error

Plugin error model shared by host and guests.

Capability failures serialize as [`PluginError`](error.md#bookclerk_plugin_abierrorpluginerror) in the Cap'n Proto
`PluginError` struct (or the equivalent workerd reject payload). Codes are
stable `snake_case` strings matching the `PluginErrorCode` enum in
`schema/plugin.capnp`. Unknown future codes are preserved as
`PluginErrorCode::Unknown` plus the raw wire string — they are never
collapsed to `PluginErrorCode::Internal`.



## Module: generated

GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp.

Rust projection of the `rust-generated` section of `plugin.capnp`: owned
DTOs for every typed method payload plus `write_*` / `read_*` codecs over
the capnpc builders and readers. `$optional` scalars are `Option<T>`
(wire zero value = `None`); `{ ok, err }` reply unions are
`Result<T, PluginError>`.



## Module: guest_sql

Host-side grammar and scope checks for guest-authored typed SQL.

Overwriting [`crate::DbPlanStatementKind`](db_execute.md#bookclerk_plugin_abidb_executedbplanstatementkind) is classification, not
authorization. Guests may only run canonical DML/SELECT against allowed
tables, with bind counts and result-selection fields that match the
statement. Host-authored schema batches must not use this path.



## Module: json_bytes

Serde helpers projecting Cap'n Proto `Data` fields as base64 text in JSON.

The typed ABI travels as Cap'n Proto; JSON projections exist only for
transport-private bridges (the workerd HTTP bridge), operator
diagnostics, and tests. Bytes serialize as standard base64 (padded) and
deserialize from either base64 text or a JSON number array, so hand-written
fixtures stay readable.



## Module: kind

Output-plugin configuration DTOs carried as `application/json` inside
[`crate::ExtensibleConfig`](rpc_types.md#bookclerk_plugin_abirpc_typesextensibleconfig) (`Bindings.config` for the `storage` entrypoint).

Field names serialize as **camelCase** on the wire. Every method payload
is a typed Cap'n Proto struct generated into [`crate::generated`](generated.md); only the
operator-configured destination knobs remain JSON because they are
plugin-private configuration, not ABI method parameters.



## Module: methods

Role capability method names (camelCase on the wire).

Each submodule exposes a single [`NAME`](methods/health.md#bookclerk_plugin_abimethodshealthname) constant equal to
the corresponding `ContentSource` / `RemoteLibrary` / `Oidc` / `Cli` /
`EventConsumer` / `EventPublisher` / `QueueConsumer` / `QueueProducer` /
`PluginWorker` method in
`schema/plugin.capnp`. Prefer these constants over string literals when
dispatching or documenting capabilities.

[`METHOD_NAMES`](methods.md#bookclerk_plugin_abimethodsmethod_names) enumerates every known method for discovery / schema
drift tests. Flat `UPPER_SNAKE` aliases live under `names`.



## Module: package_version

`plugin.toml` `version` as a semantic version.

Every plugin declares a semver `version`. The host pins the `[[settings]]`
schema per plugin key together with the version that produced it, and
orders versions to decide whether a host may re-pin (newer), must refuse
to spawn (older), or is unchanged.



## Module: plugin_capnp

Generated Cap'n Proto RPC interfaces (`schema/plugin.capnp`).

Included at crate root because `capnpc` emits `crate::plugin_capnp` paths.



## Module: plugin_host_capnp

Host-private Cap'n Proto RPC interfaces (`schema/plugin_host.capnp`).



## Module: settings_schema

Operator-facing plugin settings schema (`[[settings]]` in `plugin.toml`).

A plugin declares the keys an operator may set on one of its instances,
their types, and their defaults. The host validates instance documents
against this schema and renders it in Settings; it hard-codes nothing per
plugin. The same struct is the `plugin.toml` TOML shape and the JSON body
pinned in the library database at enrollment.



## Module: sql_desugar

Host semantic desugars for canonical Bookclerk SQL.

These rewrites are **backend-independent** and define Bookclerk behavior:
unspecified `ORDER BY` matches SQLite NULL ordering, and `/` `%` by zero
yield NULL via portable `NULLIF`. Adapters must not repeat them (doing so
would double-wrap `NULLIF` and shift proof spans).



## Module: sql_types

Bookclerk SQL v1 types, CAST matrix, and fail-closed expression checking.

Schema metadata for plugin-owned bindings is durable in
[`SQL_CATALOG_TABLE`](sql_types.md#bookclerk_plugin_abisql_typessql_catalog_table). This module only interprets canonical SQL; adapters
persist catalog rows at execute.



