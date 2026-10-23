# bookclerk_library::operator_token

Durable operator API token in `encrypted_secrets`.

Resolution for daemon/CLI (when `[daemon.auth].enabled`):
1. `BOOKCLERK_OPERATOR_TOKEN` env override (not written to DB)
2. This sealed row
3. One-time import from a legacy `operator.token` file (then delete the file)
4. Mint a new token and seal it here

Payload is UTF-8 plaintext of the token string (sealed-v1 via process DEK).

| Item | Kind | Summary |
| --- | --- | --- |
| [`ResolveOperatorToken`](#resolveoperatortoken) | enum | How an operator token was resolved. |
| [`env_operator_token`](#env_operator_token) | function | Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty. |
| [`load_operator_token`](#load_operator_token) | function | Load the operator token from `encrypted_secrets`, if present. |
| [`save_operator_token`](#save_operator_token) | function | Persist an operator token into `encrypted_secrets` (sealed-v1). |
| [`rotate_operator_token`](#rotate_operator_token) | function | Mint a new operator token, store it, and return it. |
| [`resolve_operator_token`](#resolve_operator_token) | function | Resolve the effective operator token for an enabled daemon auth config. |
| [`read_or_create_operator_token`](#read_or_create_operator_token) | function | Read-or-create for daemon startup when auth is enabled. |
| [`legacy_operator_token_file`](#legacy_operator_token_file) | function | Best-effort check whether a path looks like a leftover token file (tests/docs). |
| [`OPERATOR_TOKEN_SECRET_NAME`](#operator_token_secret_name) | constant | Canonical secret name for the operator API token. |
| [`OPERATOR_TOKEN_ACCOUNT_ID`](#operator_token_account_id) | constant | Canonical `account_id` for the operator API token row. |

## Enums

### ResolveOperatorToken

```rust
pub enum ResolveOperatorToken {
    Env,
    Database,
    LegacyFile,
    Generated,
}
```

How an operator token was resolved.

**Variants**

- `Env`: Value from `BOOKCLERK_OPERATOR_TOKEN`.
- `Database`: Value from `encrypted_secrets`.
- `LegacyFile`: Imported from a legacy on-disk token file into the database.
- `Generated`: Freshly minted and stored in the database.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### env_operator_token

```rust
pub fn env_operator_token() -> Result<Option<String>>
```

Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty.

#### Errors

Returns an error when the operation fails.

### load_operator_token

```rust
pub async fn load_operator_token(db: &DatabaseConnection) -> Result<Option<String>>
```

Load the operator token from `encrypted_secrets`, if present.

#### Errors

Returns an error when the operation fails.

### save_operator_token

```rust
pub async fn save_operator_token(db: &DatabaseConnection, token: &str) -> Result<()>
```

Persist an operator token into `encrypted_secrets` (sealed-v1).

#### Errors

Returns an error when the operation fails.

### rotate_operator_token

```rust
pub async fn rotate_operator_token(db: &DatabaseConnection) -> Result<String>
```

Mint a new operator token, store it, and return it.

Refuses when `BOOKCLERK_OPERATOR_TOKEN` is set (env override would still win).

#### Errors

Returns an error when the operation fails.

### resolve_operator_token

```rust
pub async fn resolve_operator_token(
    config: &Config,
    db: &DatabaseConnection,
    create: bool,
) -> Result<Option<(String, ResolveOperatorToken)>>
```

Resolve the effective operator token for an enabled daemon auth config.

When `create` is true, mints and stores a token if none exists (daemon startup).
When `create` is false, returns `Ok(None)` if no env/DB/legacy source provides one.

#### Errors

Returns an error when the operation fails.

### read_or_create_operator_token

```rust
pub async fn read_or_create_operator_token(
    config: &Config,
    db: &DatabaseConnection,
) -> Result<(String, ResolveOperatorToken)>
```

Read-or-create for daemon startup when auth is enabled.

#### Errors

Returns an error when the operation fails.

### legacy_operator_token_file

```rust
pub fn legacy_operator_token_file(files_dir: &Path) -> PathBuf
```

Best-effort check whether a path looks like a leftover token file (tests/docs).

## Constants

### OPERATOR_TOKEN_SECRET_NAME

```rust
pub const OPERATOR_TOKEN_SECRET_NAME: &str = "default";
```

Canonical secret name for the operator API token.

### OPERATOR_TOKEN_ACCOUNT_ID

```rust
pub const OPERATOR_TOKEN_ACCOUNT_ID: &str = "default";
```

Canonical `account_id` for the operator API token row.
