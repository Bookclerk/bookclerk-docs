# bookclerk_plugin_abi::methods::send_batch

Send a batch of messages into a granted queue (Cap'n Proto
`QueueProducer.sendBatch`).

Params: a list of [`crate::QueueSendMessage`](../generated.md#queuesendmessage) plus a default delay.
Result: [`crate::QueueSendOk`](../generated.md#queuesendok).

## Constants

### NAME

```rust
pub const NAME: &str = "sendBatch";
```

Wire method name `"sendBatch"`.
