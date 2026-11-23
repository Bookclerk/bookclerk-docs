# bookclerk_plugin_destination_r2

Cloudflare R2 destination plugin.

Writes acquire output to an R2 bucket through R2's S3 API, which
enforces conditional writes and serves batch deletes and server-side
copies, and follows the bucket's changes through R2 event notifications
pulled from a Cloudflare Queue. See [`plugin`](plugin.md) for the Cap'n Proto
destination / job-handler surface, and [`provision`](provision.md) for the guided
setup that creates the bucket and its scoped token.

## Modules

- [`plugin`](plugin.md): Destination + job-handler surface for the Cloudflare R2 guest.
- [`provision`](provision.md): The R2 destination's guided setup: with the operator's Cloudflare API token it creates (or reuses) the bucket and mints an API token scoped to that one bucket, whose id and hashed value are the S3 credentials the destination then uses.

## Constants

### ID

```rust
pub const ID: &str = "r2";
```

Plugin id advertised in describe() and the manifest.
