# bookclerk_db_testkit::binding

A plugin-owned database that behaves like a granted `[[databases]]`
binding on a deployed host.

A plugin's `env.DB` reaches an isolated unit through the host, which
authorizes the SQL against the binding-owned scope, wraps every batch in
a receipt and holds it to the adapter's advertisement. The in-process
fakes an author would otherwise test against do none of that.
[`BindingDatabase`](#bindingdatabase) runs the same host path on an in-memory SQLite unit
under one [`Profile`](../bookclerk_library/test_support.md#profile), so a plugin's queries meet the caps, the typed
`limit_exceeded` error and the refused interactive transaction before
they meet a real adapter.

The TypeScript and Python SDKs reach it through the `bookclerk-db-testkit`
binary, which serves one binding over the granted channel the isolates
already speak (`POST /db/execute`, `POST /db/execute-streamed`,
`GET /db/limits`).

## Structs

### BindingDatabase

```rust
pub struct BindingDatabase {
    /* private fields */
}
```

An in-memory plugin database binding under one capability profile.

Open one with [`BindingDatabase::open`](#bindingdatabaseopen), giving it the plugin's
`databaseMigrations` registration, and run the plugin's SQL through the
[`GuestDatabase`](../bookclerk_plugin_abi/README.md#guestdatabase) methods the host serves.

#### BindingDatabase::open

```rust
pub async fn open(
    profile: Profile,
    migrations: Vec<PluginMigration>,
) -> Result<Self, PluginError>
```

Opens an empty in-memory unit under `profile` and applies
`migrations` the way the host applies a plugin's
`databaseMigrations` registration at enrollment.

##### Errors

Returns `invalid_params` when the registration fails the BookclerkSQL
proof (an unsupported statement, a reserved table, a repeated id),
and the engine's error when the unit cannot be opened or a migration
cannot be applied.

#### BindingDatabase::profile

```rust
pub fn profile(&self) -> Profile
```

The profile this binding enforces.

#### BindingDatabase::capabilities

```rust
pub fn capabilities(&self) -> &DbCapabilities
```

The advertisement every batch is held to.

#### BindingDatabase::connection

```rust
pub fn connection(&self) -> &DatabaseConnection
```

The unit itself, for a test that inspects what the plugin wrote.

#### BindingDatabase::batch_sizes

```rust
pub fn batch_sizes(&self) -> Vec<usize>
```

Statement counts of every batch sent so far, in arrival order,
including batches the profile turned away.

Implements: `Debug`, `GuestDatabase`
