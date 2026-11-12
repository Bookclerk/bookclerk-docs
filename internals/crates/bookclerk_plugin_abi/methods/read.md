# bookclerk_plugin_abi::methods::read

Read one page of past outbox events (Cap'n Proto `EventPublisher.read`).

Params: [`crate::EventReadRequest`](../event_read.md#eventreadrequest). Result: [`crate::EventReadPage`](../event_read.md#eventreadpage).

## Constants

### NAME

```rust
pub const NAME: &str = "read";
```

Wire method name `"read"`.
