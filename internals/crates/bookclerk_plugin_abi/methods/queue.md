# bookclerk_plugin_abi::methods::queue

Deliver one ordered batch of queue messages (Cap'n Proto `QueueConsumer.queue`).

Params: [`crate::QueueBatch`](../generated.md#queuebatch). Result: one [`crate::QueueResult`](../generated.md#queueresult) per message.

## Constants

### NAME

```rust
pub const NAME: &str = "queue";
```

Wire method name `"queue"`.
