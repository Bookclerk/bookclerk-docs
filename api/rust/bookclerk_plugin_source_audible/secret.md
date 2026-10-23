**bookclerk_plugin_source_audible > secret**

# Module: secret

## Contents

**Functions**

- [`resolve_auth_password`](#resolve_auth_password) - Resolve the auth encryption passphrase from `BOOKCLERK_AUTH_PASSWORD`, if set.

**Constants**

- [`AUTH_PASSWORD_ENV`](#auth_password_env) - Environment variable holding the auth encryption passphrase.

---

## bookclerk_plugin_source_audible::secret::AUTH_PASSWORD_ENV

*Constant*: `&str`

Environment variable holding the auth encryption passphrase.

For new installs this is optional — the process DEK (`master.key`) is
sufficient. Set it to wrap `master.key` at rest (strongly recommended for
production) and to read legacy `json-encrypted` DB rows.



## bookclerk_plugin_source_audible::secret::resolve_auth_password

*Function*

Resolve the auth encryption passphrase from `BOOKCLERK_AUTH_PASSWORD`, if set.

Used for reading legacy `json-encrypted` rows and loading external
audible-rs files that were previously encrypted with this passphrase.

# Errors

Returns an error when the operation fails.

```rust
fn resolve_auth_password() -> crate::error::Result<Option<secrecy::SecretString>>
```



