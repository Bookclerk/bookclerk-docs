**bookclerk_library > operator_token**

# Module: operator_token

## Contents

**Enums**

- [`ResolveOperatorToken`](#resolveoperatortoken) - How an operator token was resolved.

**Functions**

- [`env_operator_token`](#env_operator_token) - Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty.
- [`legacy_operator_token_file`](#legacy_operator_token_file) - Best-effort check whether a path looks like a leftover token file (tests/docs).
- [`load_operator_token`](#load_operator_token) - Load the operator token from `encrypted_secrets`, if present.
- [`read_or_create_operator_token`](#read_or_create_operator_token) - Read-or-create for daemon startup when auth is enabled.
- [`resolve_operator_token`](#resolve_operator_token) - Resolve the effective operator token for an enabled daemon auth config.
- [`rotate_operator_token`](#rotate_operator_token) - Mint a new operator token, store it, and return it.
- [`save_operator_token`](#save_operator_token) - Persist an operator token into `encrypted_secrets` (sealed-v1).

**Constants**

- [`OPERATOR_TOKEN_ACCOUNT_ID`](#operator_token_account_id) - Canonical `account_id` for the operator API token row.
- [`OPERATOR_TOKEN_SECRET_NAME`](#operator_token_secret_name) - Canonical secret name for the operator API token.

---

## bookclerk_library::operator_token::OPERATOR_TOKEN_ACCOUNT_ID

*Constant*: `&str`

Canonical `account_id` for the operator API token row.



## bookclerk_library::operator_token::OPERATOR_TOKEN_SECRET_NAME

*Constant*: `&str`

Canonical secret name for the operator API token.



## bookclerk_library::operator_token::ResolveOperatorToken

*Enum*

How an operator token was resolved.

**Variants:**
- `Env` - Value from `BOOKCLERK_OPERATOR_TOKEN`.
- `Database` - Value from `encrypted_secrets`.
- `LegacyFile` - Imported from a legacy on-disk token file into the database.
- `Generated` - Freshly minted and stored in the database.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ResolveOperatorToken`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ResolveOperatorToken) -> bool`



## bookclerk_library::operator_token::env_operator_token

*Function*

Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty.

# Errors

Returns an error when the operation fails.

```rust
fn env_operator_token() -> crate::error::Result<Option<String>>
```



## bookclerk_library::operator_token::legacy_operator_token_file

*Function*

Best-effort check whether a path looks like a leftover token file (tests/docs).

```rust
fn legacy_operator_token_file(files_dir: &std::path::Path) -> std::path::PathBuf
```



## bookclerk_library::operator_token::load_operator_token

*Function*

Load the operator token from `encrypted_secrets`, if present.

# Errors

Returns an error when the operation fails.

```rust
fn load_operator_token(db: &sea_orm::DatabaseConnection) -> crate::error::Result<Option<String>>
```



## bookclerk_library::operator_token::read_or_create_operator_token

*Function*

Read-or-create for daemon startup when auth is enabled.

# Errors

Returns an error when the operation fails.

```rust
fn read_or_create_operator_token(config: &bookclerk_config::Config, db: &sea_orm::DatabaseConnection) -> crate::error::Result<(String, ResolveOperatorToken)>
```



## bookclerk_library::operator_token::resolve_operator_token

*Function*

Resolve the effective operator token for an enabled daemon auth config.

When `create` is true, mints and stores a token if none exists (daemon startup).
When `create` is false, returns `Ok(None)` if no env/DB/legacy source provides one.

# Errors

Returns an error when the operation fails.

```rust
fn resolve_operator_token(config: &bookclerk_config::Config, db: &sea_orm::DatabaseConnection, create: bool) -> crate::error::Result<Option<(String, ResolveOperatorToken)>>
```



## bookclerk_library::operator_token::rotate_operator_token

*Function*

Mint a new operator token, store it, and return it.

Refuses when `BOOKCLERK_OPERATOR_TOKEN` is set (env override would still win).

# Errors

Returns an error when the operation fails.

```rust
fn rotate_operator_token(db: &sea_orm::DatabaseConnection) -> crate::error::Result<String>
```



## bookclerk_library::operator_token::save_operator_token

*Function*

Persist an operator token into `encrypted_secrets` (sealed-v1).

# Errors

Returns an error when the operation fails.

```rust
fn save_operator_token(db: &sea_orm::DatabaseConnection, token: &str) -> crate::error::Result<()>
```



