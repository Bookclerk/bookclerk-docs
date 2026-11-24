# bookclerk_plugin_database_d1::provision

The D1 database's guided setup: with the operator's Cloudflare API token
it finds (or creates) the library's D1 database and mints an account
token that can use D1 and nothing else, for the host to keep.

The operator's token needs "D1 Edit" (to find and create the database)
and "Account API Tokens Edit" (to mint and delete the library's token).
It is used for this run only. The minted token is returned as the
`api_token` secret; `bookclerk config database setup d1` seals it with the
library's key and keeps it in the same store as the master key.

A second run keeps the token the first run minted (its id comes back as
the `api_token_id` setting): shared branches of the library hold a copy,
and a new token would cut them off. With `rotate_token` it mints a fresh
token and deletes the earlier one, so tokens never pile up. The setup
never deletes a database: it holds the library. The replies are read by
[`bookclerk_storage::cloudflare_api`](../bookclerk_storage/cloudflare_api.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`D1Provisioning`](#d1provisioning) | struct | The `provisioning` entrypoint of the D1 guest. |
| [`Jurisdiction`](#jurisdiction) | enum | Where a new D1 database's data is kept. |
| [`API_BASE`](#api_base) | constant | The Cloudflare API the setup calls. |
| [`DEFAULT_DATABASE_NAME`](#default_database_name) | constant | The database name used when the operator chooses none. |

## Structs

### D1Provisioning

```rust
pub struct D1Provisioning {
    /* private fields */
}
```

The `provisioning` entrypoint of the D1 guest.

#### D1Provisioning::new

```rust
pub fn new(api_base: &str, config: Value) -> Result<Self, PluginError>
```

The entrypoint for a setup whose settings are `config`.

##### Errors

invalid_params when `config` is not an object of the expected shape.

Implements: `Provisioning`

## Enums

### Jurisdiction

```rust
pub enum Jurisdiction {
    Default,
    Eu,
    Fedramp,
}
```

Where a new D1 database's data is kept.

**Variants**

- `Default`: Wherever Cloudflare places it.
- `Eu`: Inside the European Union.
- `Fedramp`: FedRAMP-compliant data centers.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`

## Constants

### API_BASE

```rust
pub const API_BASE: &str = "https://api.cloudflare.com/client/v4";
```

The Cloudflare API the setup calls.

### DEFAULT_DATABASE_NAME

```rust
pub const DEFAULT_DATABASE_NAME: &str = "bookclerk";
```

The database name used when the operator chooses none.
