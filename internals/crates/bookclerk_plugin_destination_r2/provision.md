# bookclerk_plugin_destination_r2::provision

The R2 destination's guided setup: with the operator's Cloudflare API
token it creates (or reuses) the bucket and mints an API token scoped to
that one bucket, whose id and hashed value are the S3 credentials the
destination then uses.

The operator's token needs "Workers R2 Storage Edit" (to create the
bucket) and "Account API Tokens Edit" (to mint and delete the bucket's
token). It is used for this run only; the destination keeps only the
bucket token's S3 credentials, which can read and write that bucket's
objects and nothing else.

A second run mints a fresh bucket token and deletes the one the first
run minted, so a re-run rotates the credentials rather than piling up
tokens. Deprovisioning deletes the bucket token and leaves the bucket
and its books alone. The replies are read by
[`bookclerk_storage::cloudflare_api`](../bookclerk_storage/cloudflare_api.md).

| Item | Kind | Summary |
| --- | --- | --- |
| [`R2Provisioning`](#r2provisioning) | struct | The `provisioning` entrypoint of the R2 guest. |
| [`API_BASE`](#api_base) | constant | The Cloudflare API the setup calls. |

## Structs

### R2Provisioning

```rust
pub struct R2Provisioning {
    /* private fields */
}
```

The `provisioning` entrypoint of the R2 guest.

#### R2Provisioning::new

```rust
pub fn new(api_base: &str, config: Value, secrets: Value) -> Result<Self, PluginError>
```

The entrypoint for an instance whose `CONFIG` and `SECRETS` are
`config` and `secrets`.

##### Errors

invalid_params when either is not an object of the expected shape.

Implements: `Provisioning`

## Constants

### API_BASE

```rust
pub const API_BASE: &str = "https://api.cloudflare.com/client/v4";
```

The Cloudflare API the setup calls.
