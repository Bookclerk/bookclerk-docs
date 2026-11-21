# bookclerk_plugin_abi::methods::send

Send one message into a granted queue (Cap'n Proto `QueueProducer.send`),
or deliver one notification to one destination (Cap'n Proto
`Notifier.send`).

Params: [`crate::QueueSendMessage`](../generated.md#queuesendmessage) / [`crate::NotifySendParams`](../generated.md#notifysendparams).
Result: [`crate::QueueSendOk`](../generated.md#queuesendok) / [`crate::NotifySent`](../generated.md#notifysent).

## Constants

### NAME

```rust
pub const NAME: &str = "send";
```

Wire method name `"send"`.
