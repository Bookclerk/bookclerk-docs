**bookclerk_library**

# Module: bookclerk_library

## Contents

**Modules**

- [`control_plane`](#control_plane) - Database-backed control plane: node identity and typed configuration.
- [`email`](#email) - Contact-email normalization and library-backed validation.
- [`entities`](#entities) - SeaORM entities mirroring the greenfield schema in [`crate::migrations`](migrations.md).
- [`master_key`](#master_key) - Data-encryption key (DEK) for `encrypted_secrets`.
- [`migrations`](#migrations) - Host library schema: frozen plan steps plus the unreleased development pack.
- [`operator_token`](#operator_token) - Durable operator API token in `encrypted_secrets`.
- [`password`](#password) - Argon2id password hashing for first-party local login.
- [`proxy_txn`](#proxy_txn) - Fail-closed transaction injects re-exported from [`bookclerk_db_exec`](../bookclerk_db_exec/bookclerk_db_exec.md).
- [`scope`](#scope) - Plugin-scoped library access.
- [`secrets`](#secrets) - Encrypted secrets store — DB-backed credential storage.
- [`sql_plan`](#sql_plan) - Host-owned generic SQL atomic plans for database adapters.

---

## Module: control_plane

Database-backed control plane: node identity and typed configuration.

Bootstrap material (database target, `master.key`, `node-identity.json`) is
read before these rows exist. After enrollment, `core.events` is the
authority for `[events]` and `core.output` for the shared `[output]`
table. Other `config.toml` sections stay transitional.

See `docs/adr/control-plane.md`.



## Module: email

Contact-email normalization and library-backed validation.



## Module: entities

SeaORM entities mirroring the greenfield schema in [`crate::migrations`](migrations.md).

One `DeriveEntityModel` per table. Column Rust types match the proxy/native
backends: integer columns are `i64`, reals `f64`, blobs `Vec<u8>`, and text
(including RFC 3339 timestamps) `String`. Timestamps stay as `String` to
match the `TEXT` columns without relying on SeaORM's proxy chrono decoding;
[`LibraryStore`](store.md#bookclerk_librarystorelibrarystore) parses them into `chrono::DateTime<Utc>`
at the record boundary and uses these entities (`Entity::find`,
`ActiveModel`) for the majority of CRUD.



## Module: master_key

Data-encryption key (DEK) for `encrypted_secrets`.

# Design

Secrets in the DB are sealed with a **random 32-byte DEK** + per-row
XChaCha20-Poly1305 nonce (format `sealed-v1`). The DEK is **not** derived
per secret (no Argon2 on every load/save).

## Bootstrap (outside the DB)

| Source | Behavior |
| --- | --- |
| `BOOKCLERK_AUTH_PASSWORD` or `[auth].password` | Wraps/unwraps the DEK in `{files_dir}/master.key` |
| Auto-mint | If neither password nor file exists, generates a DEK and writes `master.key` (mode `0600`) |

A later password can wrap an existing `BCK1` file in place (CLI
`config master-key wrap`, `config set auth.password`, or daemon config
reload). The operator API token is **independent** and short-lived for
HTTP auth only. Rotating it does **not** re-encrypt library secrets.

Prefer a single opaque ciphertext blob per secret (not per-field columns):
Audible envelopes are already opaque, updates stay atomic, and one unseal
recovers the whole credential.



## Module: migrations

Host library schema: frozen plan steps plus the unreleased development pack.

Bookclerk has **not** frozen a production v1 schema. [`host_migration_plan`](migrations.md#bookclerk_librarymigrationshost_migration_plan)
is empty until a release cut. Live schema lives in [`unreleased_ops`](migrations.md#bookclerk_librarymigrationsunreleased_ops) (already
separated [`MigrationOp`](migrations.md#bookclerk_librarymigrationsmigrationop)s). Fresh databases apply those ops (frozen ups +
unreleased) and persist `crate::SchemaState::Unreleased` with `base_version`
equal to [`SCHEMA_VERSION`](migrations.md#bookclerk_librarymigrationsschema_version) (`0` today: no frozen revisions). Fresh init
records each frozen plan step's checksum before the unreleased marker; when
the unreleased bucket is empty the database ends
`crate::SchemaState::Frozen`. Joined SQL ([`unreleased_sql`](migrations.md#bookclerk_librarymigrationsunreleased_sql),
[`current_canonical_schema`](migrations.md#bookclerk_librarymigrationscurrent_canonical_schema)) is derived for diagnostics/export/tests.
Adapters lower canonical DDL at the execution edge
([`bookclerk_db_exec::expand_host_schema_batch`](../bookclerk_db_exec/schema_postgres.md#bookclerk_db_execschema_postgresexpand_host_schema_batch)).



## Module: operator_token

Durable operator API token in `encrypted_secrets`.

Resolution for daemon/CLI (when `[daemon.auth].enabled`):
1. `BOOKCLERK_OPERATOR_TOKEN` env override (not written to DB)
2. This sealed row
3. One-time import from a legacy `operator.token` file (then delete the file)
4. Mint a new token and seal it here

Payload is UTF-8 plaintext of the token string (sealed-v1 via process DEK).



## Module: password

Argon2id password hashing for first-party local login.



## Module: proxy_txn

Fail-closed transaction injects re-exported from [`bookclerk_db_exec`](../bookclerk_db_exec/bookclerk_db_exec.md).



## Module: scope

Plugin-scoped library access.

First-party and third-party content sources share the same boundary:
a [`SourceScope`](scope.md#bookclerk_libraryscopesourcescope) can only read/write rows whose `plugin_key` / `provider`
matches the provenance-qualified PluginKey it was opened for. Rows are not
tied to a plugin instance: every instance of that plugin, on any host,
sees the same accounts, credentials, and books. Host code that needs the
full store keeps its own [`LibraryStore`](store.md#bookclerk_librarystorelibrarystore) handle — this type does not
expose one.



## Module: secrets

Encrypted secrets store — DB-backed credential storage.

# Overview

Secrets are kept in the `encrypted_secrets` table (M10 migration). Each row
stores opaque `ciphertext` with column-level metadata describing which KDF
and cipher were used, so every secret carries its own self-contained
decryption recipe.

## Algorithms

- **New writes**: `sealed-v1` — XChaCha20-Poly1305 with the process DEK
  (from `master.key`). No per-row Argon2. No plaintext writes ever.
- **Legacy reads**: `json-encrypted` (Argon2id + BOOKCLERK_AUTH_PASSWORD)
  still decrypted. `json` (plaintext) is migrated on read if master key
  is available, otherwise rejected.

## Unseal cache

[`unseal_secret`](secrets.md#bookclerk_librarysecretsunseal_secret) keeps a process-wide plaintext cache (identity +
ciphertext fingerprint). Content-source plugins should just call
load/unseal normally — repeated acquire/scan loads stay cheap without
per-plugin caches. Upsert/delete invalidate the matching entries.

## Bootstrap secrets (NOT stored here)

`BOOKCLERK_AUTH_PASSWORD`, `BOOKCLERK_DATABASE_POSTGRES_URL`, and
`BOOKCLERK_D1_API_TOKEN` / `CLOUDFLARE_API_TOKEN` are **env-only bootstrap**
credentials. They are needed to open the DB or bootstrap the master key and
cannot be stored here. `config.toml` also stays on disk.

`BOOKCLERK_OPERATOR_TOKEN` is an optional **env override** for the durable
operator token row (`kind = operator_token`); see [`crate::operator_token`](operator_token.md).



## Module: sql_plan

Host-owned generic SQL atomic plans for database adapters.
Host-owned SQL atomic plans for database plugins.

The library compiles Bookclerk domain operations into a typed
[`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest) plus host-only
[`crate::sql_plan::host_ir::AtomicSelection`](sql_plan/host_ir.md#bookclerk_librarysql_planhost_iratomicselection) indexes. Database guests
execute the statements as one transaction and return a typed
[`bookclerk_plugin_abi::ExecuteReply`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply).



