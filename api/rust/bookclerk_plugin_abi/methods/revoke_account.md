# bookclerk_plugin_abi::methods::revoke_account

Undo store-side state for one account before the host deletes its
stored credentials (for example, deregister the login device).

Params: [`crate::RevokeAccountParams`](../generated.md#revokeaccountparams). Result: none. Best effort: the host
revokes the account whether or not the guest succeeds.

## Constants

### NAME

```rust
pub const NAME: &str = "revokeAccount";
```

Wire method name `"revokeAccount"`.
