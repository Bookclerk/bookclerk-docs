# bookclerk_plugin_destination_s3

S3 / MinIO destination plugin shared constants.

Optional destination that writes acquire output to a bucket configured via
`[output.s3]` / `BOOKCLERK_OUTPUT_S3_*`. See [`plugin`](plugin.md) for the Cap'n Proto
destination / job-handler surface.

## Modules

- [`plugin`](plugin.md): Destination + job-handler surface for the S3 / MinIO guest.

## Constants

### ID

```rust
pub const ID: &str = "s3";
```

Plugin id string advertised in describe() and config (`[output.s3]`).
