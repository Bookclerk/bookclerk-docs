# bookclerk_plugin_abi::methods::send

Send one message into a granted queue (Cap'n Proto `QueueProducer.send`).

Params: [`crate::QueueSendMessage`](../generated.md#queuesendmessage). Result: [`crate::QueueSendOk`](../generated.md#queuesendok).

## Constants

### NAME

```rust
pub const NAME: &str = "send";
```

Wire method name `"send"`.
