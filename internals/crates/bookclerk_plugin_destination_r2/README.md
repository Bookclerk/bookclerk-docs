# bookclerk_plugin_destination_r2

Cloudflare R2 destination plugin.

Writes acquire output to an R2 bucket through R2's S3 API, which
enforces conditional writes and serves batch deletes and server-side
copies, and follows the bucket's changes through R2 event notifications
pulled from a Cloudflare Queue. See [`plugin`](plugin.md) for the Cap'n Proto
destination / job-handler surface.

## Modules

- [`plugin`](plugin.md): Destination + job-handler surface for the Cloudflare R2 guest.

## Constants

### ID

```rust
pub const ID: &str = "r2";
```

Plugin id advertised in describe() and the manifest.
