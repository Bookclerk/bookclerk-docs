# bookclerk_plugin_abi::methods::check_deliverability

Run the operator's deliverability check on a notifier (Cap'n Proto
`Notifier.checkDeliverability`).

Params: [`crate::NotifyDeliverabilityParams`](../generated.md#notifydeliverabilityparams). Result:
[`crate::NotifyDeliverabilityReport`](../generated.md#notifydeliverabilityreport).

## Constants

### NAME

```rust
pub const NAME: &str = "checkDeliverability";
```

Wire method name `"checkDeliverability"`.
