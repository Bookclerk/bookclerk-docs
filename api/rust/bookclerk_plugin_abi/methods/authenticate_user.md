# bookclerk_plugin_abi::methods::authenticate_user

Validate a username/password against a remote IdP (Cap'n Proto
`Oidc.authenticateUser`) and return an external user.

Params: [`crate::AuthenticateUserParams`](../generated.md#authenticateuserparams).

## Constants

### NAME

```rust
pub const NAME: &str = "authenticateUser";
```

Wire method name `"authenticateUser"`.
