# bookclerk_plugin_destination_r2::plugin

Destination + job-handler surface for the Cloudflare R2 guest.

| Item | Kind | Summary |
| --- | --- | --- |
| [`R2Root`](#r2root) | struct | Root capability for the R2 destination guest. |
| [`Jurisdiction`](#jurisdiction) | enum | The jurisdiction an R2 bucket was created in, which picks its endpoint. |
| [`destination_from`](#destination_from) | function | Builds the destination from the instance's `CONFIG` and `SECRETS`. |

## Structs

### R2Root

```rust
pub struct R2Root;
```

Root capability for the R2 destination guest.

Implements: `PluginWorker`

## Enums

### Jurisdiction

```rust
pub enum Jurisdiction {
    Default,
    Eu,
    Fedramp,
}
```

The jurisdiction an R2 bucket was created in, which picks its endpoint.

**Variants**

- `Default`: No jurisdiction: `<account>.r2.cloudflarestorage.com`.
- `Eu`: The European Union: `<account>.eu.r2.cloudflarestorage.com`.
- `Fedramp`: FedRAMP: `<account>.fedramp.r2.cloudflarestorage.com`.

#### Jurisdiction::endpoint

```rust
pub fn endpoint(self, account: &CloudflareAccountId) -> String
```

The bucket's S3 API endpoint for `account`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`

## Functions

### destination_from

```rust
pub async fn destination_from(
    config: &ExtensibleConfig,
    secrets: &ExtensibleConfig,
) -> Result<ObjectStoreDestination, PluginError>
```

Builds the destination from the instance's `CONFIG` and `SECRETS`.

#### Arguments

* `config` - the instance settings.
* `secrets` - the instance's sealed secrets.

#### Errors

Returns invalid_params for a missing or malformed setting or secret, and
internal when the client cannot be constructed.
