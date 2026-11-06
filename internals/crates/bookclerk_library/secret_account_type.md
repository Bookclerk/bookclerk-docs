# bookclerk_library::secret_account_type

Ownership namespace for `encrypted_secrets.account_type`.

Rows tagged `integration` are store or portal credentials tied to a user
account; they are purged when that account is revoked. Rows tagged
`operator` are destination / control-plane secrets (e.g. S3 keys) that
outlive any individual store account and must never be touched by
[`delete_secrets_for_account`](secrets.md#delete_secrets_for_account).

| Item | Kind | Summary |
| --- | --- | --- |
| [`INTEGRATION`](#integration) | constant | Store / portal integration accounts (Audible, Libro, Chirp, GA, Widevine). |
| [`OPERATOR`](#operator) | constant | Operator-owned destination / control-plane secrets (S3, …). |
| [`USER`](#user) | constant | First-party user secrets (TOTP), purged when that user is deleted. |
| [`PLUGIN_INSTANCE`](#plugin_instance) | constant | Secrets owned by one `crate::control_plane::PluginInstanceId`. |

## Constants

### INTEGRATION

```rust
pub const INTEGRATION: &str = "integration";
```

Store / portal integration accounts (Audible, Libro, Chirp, GA, Widevine).

### OPERATOR

```rust
pub const OPERATOR: &str = "operator";
```

Operator-owned destination / control-plane secrets (S3, …).

### USER

```rust
pub const USER: &str = "user";
```

First-party user secrets (TOTP), purged when that user is deleted.

### PLUGIN_INSTANCE

```rust
pub const PLUGIN_INSTANCE: &str = "plugin_instance";
```

Secrets owned by one [`crate::control_plane::PluginInstanceId`](control_plane.md#plugininstanceid).

Not purged with store accounts. `account_id` is the instance id.
