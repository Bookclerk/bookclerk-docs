# bookclerk_plugin_abi::methods::fetch_covers

Cover image bodies streamed into a host sink (Cap'n Proto
`CoverFetcher.fetchCovers`).

Params: [`crate::CoverFetchParams`](../generated.md#coverfetchparams) plus a host-served
[`crate::Destination`](../README.md#destination). Result: [`crate::CoverImages`](../generated.md#coverimages).

## Constants

### NAME

```rust
pub const NAME: &str = "fetchCovers";
```

Wire method name `"fetchCovers"`.
