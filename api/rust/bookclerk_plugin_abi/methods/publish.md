# bookclerk_plugin_abi::methods::publish

Append one guest event to the host outbox (Cap'n Proto `EventPublisher.publish`).

Params: [`crate::PluginEvent`](../generated.md#pluginevent). Result: [`crate::PublishOk`](../generated.md#publishok).

## Constants

### NAME

```rust
pub const NAME: &str = "publish";
```

Wire method name `"publish"`.
