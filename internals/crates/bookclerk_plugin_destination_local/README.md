# bookclerk_plugin_destination_local

Local filesystem destination plugin shared constants.

The platform `local` guest writes acquire output under
the instance `root` setting (host-passed as `BOOKCLERK_OUTPUT_LOCAL_ROOT`). See [`plugin`](plugin.md) for the
Cap'n Proto destination / job-handler surface.

## Modules

- [`plugin`](plugin.md): Destination + job-handler surface for the local filesystem guest.

## Constants

### ID

```rust
pub const ID: &str = "local";
```

Plugin id string advertised in describe() and config (`[output.local]`).
