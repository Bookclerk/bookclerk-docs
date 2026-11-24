# bookclerk_plugin_abi::methods::download_media

Cover image bodies streamed into a host sink (Cap'n Proto
`MediaDownloader.downloadMedia`).

Params: [`crate::MediaDownloadParams`](../generated.md#mediadownloadparams) plus a host-served
[`crate::Destination`](../README.md#destination). Result: [`crate::DownloadResults`](../generated.md#downloadresults).

## Constants

### NAME

```rust
pub const NAME: &str = "downloadMedia";
```

Wire method name `"downloadMedia"`.
