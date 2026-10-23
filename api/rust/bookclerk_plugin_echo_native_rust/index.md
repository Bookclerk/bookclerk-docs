# bookclerk_plugin_echo_native_rust

Reference Echo integration — native Rust guest.

Speaks Cap'n Proto `api_version = 3` via [`PluginWorker`]: `open` exports the
`eventConsumer` and `queueConsumer` triggers plus the `remoteLibrary` and
`cli` entrypoints.

## Modules

