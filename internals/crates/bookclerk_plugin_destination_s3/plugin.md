# bookclerk_plugin_destination_s3::plugin

Destination + job-handler surface for the S3 / MinIO guest.

| Item | Kind | Summary |
| --- | --- | --- |
| [`S3Destination`](#s3destination) | struct | S3 destination capability (multipart sink for streamed puts). |
| [`S3Root`](#s3root) | struct | Root capability for the optional S3 destination guest. |

## Structs

### S3Destination

```rust
pub struct S3Destination {
    /* private fields */
}
```

S3 destination capability (multipart sink for streamed puts).

#### S3Destination::from_config

```rust
pub async fn from_config(config: &ExtensibleConfig) -> Result<Self, PluginError>
```

Builds a destination from the `open` `Bindings::config` payload
(an [`OutputS3ContextDto`](../bookclerk_plugin_abi/kind.md#outputs3contextdto) as `application/json`).

##### Errors

Returns invalid_params when the payload is not an S3 context, or internal
when the client cannot be constructed.

#### S3Destination::from_bindings

```rust
pub async fn from_bindings(bindings: &Bindings) -> Result<Self, PluginError>
```

[`Self::from_config`](#s3destinationfrom_config) plus credentials from `SECRETS` when `CONFIG` omits them.

The host keeps access keys out of `CONFIG` and the spawn config table.

##### Errors

Returns invalid_params when the payload is not an S3 context, or internal
when the client cannot be constructed.

Implements: `Destination`

### S3Root

```rust
pub struct S3Root;
```

Root capability for the optional S3 destination guest.

Implements: `PluginWorker`
