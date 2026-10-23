**bookclerk_config > database**

# Module: database

## Contents

**Structs**

- [`DatabaseConfig`](#databaseconfig) - Top-level `[database]` — selects the active backend plugin.
- [`DatabaseD1Config`](#databased1config) - Cloudflare D1 backend (`[database.d1]`).
- [`DatabasePostgresConfig`](#databasepostgresconfig) - PostgreSQL backend (`[database.postgres]`).
- [`DatabaseSqliteConfig`](#databasesqliteconfig) - Local SQLite destination (`[database.sqlite]`).

**Enums**

- [`DatabasePluginKind`](#databasepluginkind) - Which first-party database plugin is active.

**Functions**

- [`resolve_d1_api_token`](#resolve_d1_api_token) - Resolve the D1 API token from the environment (host-mediated secret).
- [`resolve_postgres_url`](#resolve_postgres_url) - Resolve the Postgres connection URL from config (env already applied).

---

## bookclerk_config::database::DatabaseConfig

*Struct*

Top-level `[database]` — selects the active backend plugin.

**Fields:**
- `plugin: String` - Active plugin id (first-party `sqlite` / `d1` / `postgres`, or any
- `sqlite: DatabaseSqliteConfig` - Local SQLite knobs (`[database.sqlite]`).
- `d1: DatabaseD1Config` - Cloudflare D1 knobs (`[database.d1]`).
- `postgres: DatabasePostgresConfig` - PostgreSQL knobs (`[database.postgres]`).
- `plugins: toml::Table` - Opaque `[database.<id>]` tables for third-party database adapters.

**Methods:**

- `fn active_plugin(self: &Self) -> Result<DatabasePluginKind>` - Parsed first-party active plugin; `Err` for third-party adapter ids.
- `fn plugin_table(self: &Self, id: &str) -> Option<&toml::Table>` - Borrow the opaque `[database.<id>]` table for a third-party adapter.
- `fn sqlite_path(self: &Self, files_dir: &std::path::Path) -> PathBuf` - Resolve the SQLite file path (relative paths join `files_dir`).
- `fn validate(self: &Self) -> Result<()>` - Soft validation for the selected plugin (required ids / URL presence).

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DatabaseConfig) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DatabaseConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::database::DatabaseD1Config

*Struct*

Cloudflare D1 backend (`[database.d1]`).

The API token is supplied via the `BOOKCLERK_D1_API_TOKEN` or
`CLOUDFLARE_API_TOKEN` environment variable (no credentials file).

**Fields:**
- `enabled: bool` - Kept for symmetry with other plugins; ignored when another plugin is active.
- `account_id: String` - Cloudflare account id that owns the D1 database.
- `database_id: String` - D1 database UUID from the Cloudflare dashboard / API.
- `api_base: String` - Cloudflare API base (default `https://api.cloudflare.com/client/v4`).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DatabaseD1Config`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DatabaseD1Config) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::database::DatabasePluginKind

*Enum*

Which first-party database plugin is active.

**Variants:**
- `Sqlite` - Local SQLite file under the files directory (default).
- `D1` - Cloudflare D1 via the HTTP API.
- `Postgres` - PostgreSQL via a connection URL.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Stable TOML / CLI id (`sqlite`, `d1`, `postgres`).
- `fn parse(s: &str) -> Option<Self>` - Parse a first-party backend id or kind token (`sqlite`, `postgres`, `pg`, `d1`).

**Traits:** Copy, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &DatabasePluginKind) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> DatabasePluginKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DatabasePluginKind`



## bookclerk_config::database::DatabasePostgresConfig

*Struct*

PostgreSQL backend (`[database.postgres]`).

Connect via a standard Postgres connection URL:
`postgres://user:password@host:5432/dbname`

The URL contains credentials — prefer `url_file` (pointing at a secrets volume)
or the `BOOKCLERK_DATABASE_POSTGRES_URL` environment variable over embedding the
URL in TOML.

**Fields:**
- `enabled: bool` - Kept for symmetry with other plugins; ignored when another plugin is active.
- `url: Option<String>` - PostgreSQL connection URL (e.g. `postgres://user:pass@host/db`).
- `url_file: Option<std::path::PathBuf>` - Path to a file containing the connection URL (one line, trimmed).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> DatabasePostgresConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DatabasePostgresConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::database::DatabaseSqliteConfig

*Struct*

Local SQLite destination (`[database.sqlite]`).

**Fields:**
- `enabled: bool` - Kept for symmetry with other plugins; ignored when another plugin is active.
- `path: Option<std::path::PathBuf>` - DB file path (default `library.db` under the files dir).

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DatabaseSqliteConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DatabaseSqliteConfig`



## bookclerk_config::database::resolve_d1_api_token

*Function*

Resolve the D1 API token from the environment (host-mediated secret).

Uses `BOOKCLERK_D1_API_TOKEN`, falling back to `CLOUDFLARE_API_TOKEN`.

# Errors

Returns an error when the operation fails.

```rust
fn resolve_d1_api_token() -> crate::error::Result<String>
```



## bookclerk_config::database::resolve_postgres_url

*Function*

Resolve the Postgres connection URL from config (env already applied).

`url_file` takes precedence over `url`.

# Errors

Returns an error when the operation fails.

```rust
fn resolve_postgres_url(config: &crate::Config) -> crate::error::Result<String>
```



