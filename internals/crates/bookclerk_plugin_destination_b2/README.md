# bookclerk_plugin_destination_b2

Backblaze B2 destination plugin.

Writes acquire output to a B2 bucket through B2's native API, with
large files for long uploads, server-side copies and probes from the
download URL. B2 has no conditional writes, so the host checks before
each conditional write and reports that it was not atomic. See
[`plugin`](plugin.md) for the Cap'n Proto destination / job-handler surface.

## Modules

- [`plugin`](plugin.md): Destination + job-handler surface for the Backblaze B2 guest.

## Constants

### ID

```rust
pub const ID: &str = "b2";
```

Plugin id advertised in describe() and the manifest.
