# bookclerk_plugin_destination_s3::plugin

Destination + job-handler surface for the S3 / MinIO guest.

| Item | Kind | Summary |
| --- | --- | --- |
| [`S3Root`](#s3root) | struct | Root capability for the optional S3 destination guest. |
| [`destination_from_config`](#destination_from_config) | function | Builds the destination from the `open` `Bindings::config` payload (an `OutputS3ContextDto` as `application/json`). |
| [`destination_from_bindings`](#destination_from_bindings) | function | `destination_from_config` plus credentials from `SECRETS` when `CONFIG` omits them. |

## Structs

### S3Root

```rust
pub struct S3Root;
```

Root capability for the optional S3 destination guest.

Implements: `PluginWorker`

## Functions

### destination_from_config

```rust
pub async fn destination_from_config(
    config: &ExtensibleConfig,
) -> Result<ObjectStoreDestination<S3Backend>, PluginError>
```

Builds the destination from the `open` `Bindings::config` payload
(an [`OutputS3ContextDto`](../bookclerk_plugin_abi/kind.md#outputs3contextdto) as `application/json`).

#### Errors

Returns invalid_params when the payload is not an S3 context, or internal
when the client cannot be constructed.

### destination_from_bindings

```rust
pub async fn destination_from_bindings(
    bindings: &Bindings,
) -> Result<ObjectStoreDestination<S3Backend>, PluginError>
```

[`destination_from_config`](#destination_from_config) plus credentials from `SECRETS` when `CONFIG`
omits them.

The host keeps access keys out of `CONFIG` and the spawn config table.

#### Errors

Returns invalid_params when the payload is not an S3 context, or internal
when the client cannot be constructed.
