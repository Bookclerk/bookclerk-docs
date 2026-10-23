# bookclerk_plugin_abi::methods::exists

Test whether an object key exists in an output destination.

Wire name kept for consent grants; guests implement existence via
[`crate::Destination::head`](../index.md#destinationhead) (`None` means the key is missing).

## Constants

### NAME

```rust
pub const NAME: &str = "exists";
```

Wire method name `"exists"`.
