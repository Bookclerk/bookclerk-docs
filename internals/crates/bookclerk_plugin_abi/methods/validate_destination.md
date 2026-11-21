# bookclerk_plugin_abi::methods::validate_destination

Normalise and check a patron's address on a notifier (Cap'n Proto
`Notifier.validateDestination`).

Params: [`crate::NotifyValidateParams`](../generated.md#notifyvalidateparams). Result:
[`crate::NotifyValidated`](../generated.md#notifyvalidated).

## Constants

### NAME

```rust
pub const NAME: &str = "validateDestination";
```

Wire method name `"validateDestination"`.
