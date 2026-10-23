# bookclerk_library::secret_kind

Well-known `kind` values for the `encrypted_secrets` table.

Runtime credentials live here. Bootstrap credentials
(`BOOKCLERK_AUTH_PASSWORD`, `BOOKCLERK_D1_API_TOKEN`,
`BOOKCLERK_DATABASE_POSTGRES_URL`) are env-only and never stored in this
table. `BOOKCLERK_OPERATOR_TOKEN` may override the durable
[`OPERATOR_TOKEN`](#operator_token) row without writing it.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SOURCE_AUTH`](#source_auth) | constant | Store / source OAuth credentials (Audible, Libro.fm, Chirp, GA). |
| [`S3`](#s3) | constant | S3 / object-storage credentials. |
| [`WIDEVINE`](#widevine) | constant | Widevine L3 CDM device blob (`kind = widevine`). |
| [`OPERATOR_TOKEN`](#operator_token) | constant | Daemon HTTP operator API token (Bearer / GUI login). |
| [`OIDC_CLIENT`](#oidc_client) | constant | OIDC/OAuth relying-party client secret (`name` = provider id). |
| [`TOTP`](#totp) | constant | User TOTP shared secret (`name` = `primary` or `pending`). |
| [`PLUGIN_INSTANCE`](#plugin_instance) | constant | Plugin-instance secret referenced by a `plugin_instance` config document. |
| [`CLUSTER_LINK`](#cluster_link) | constant | On a host node: the private node keys of one owner-cluster link (`provider` = owner cluster id, `account_id` = this node's id there). |
| [`CLUSTER_JOIN`](#cluster_join) | constant | On a host joining a library: the PKCE verifier and node keys of one sign-in in progress (`provider` = owner cluster id, `account_id` = the pending join's state hash). Deleted when the join completes or expires. |

## Constants

### SOURCE_AUTH

```rust
pub const SOURCE_AUTH: &str = "source_auth";
```

Store / source OAuth credentials (Audible, Libro.fm, Chirp, GA).

### S3

```rust
pub const S3: &str = "s3";
```

S3 / object-storage credentials.

### WIDEVINE

```rust
pub const WIDEVINE: &str = "widevine";
```

Widevine L3 CDM device blob (`kind = widevine`).

### OPERATOR_TOKEN

```rust
pub const OPERATOR_TOKEN: &str = "operator_token";
```

Daemon HTTP operator API token (Bearer / GUI login).

### OIDC_CLIENT

```rust
pub const OIDC_CLIENT: &str = "oidc_client";
```

OIDC/OAuth relying-party client secret (`name` = provider id).

### TOTP

```rust
pub const TOTP: &str = "totp";
```

User TOTP shared secret (`name` = `primary` or `pending`).

### PLUGIN_INSTANCE

```rust
pub const PLUGIN_INSTANCE: &str = "plugin_instance";
```

Plugin-instance secret referenced by a `plugin_instance` config document.

Ciphertext stays in this table. The configuration document stores the
ref name only.

### CLUSTER_LINK

```rust
pub const CLUSTER_LINK: &str = "cluster_link";
```

On a host node: the private node keys of one owner-cluster link
(`provider` = owner cluster id, `account_id` = this node's id there).

### CLUSTER_JOIN

```rust
pub const CLUSTER_JOIN: &str = "cluster_join";
```

On a host joining a library: the PKCE verifier and node keys of one
sign-in in progress (`provider` = owner cluster id, `account_id` =
the pending join's state hash). Deleted when the join completes or
expires.
