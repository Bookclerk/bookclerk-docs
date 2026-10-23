# bookclerk_plugin_abi::methods::describe

Return the guest's declared CLI schema ([`crate::CliSchema`](../generated.md#clischema); Cap'n Proto
`PluginCli.describe`). Also the root `PluginWorker.describe` identity call.

## Constants

### NAME

```rust
pub const NAME: &str = "describe";
```

Wire method name `"describe"`.
