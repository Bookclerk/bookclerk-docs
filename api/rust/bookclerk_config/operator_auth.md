**bookclerk_config > operator_auth**

# Module: operator_auth

## Contents

**Enums**

- [`ResolveOperatorTokenEnv`](#resolveoperatortokenenv) - How an operator token was resolved from env (config-layer only).

**Functions**

- [`generate_operator_token`](#generate_operator_token) - Generate a 32-byte hex operator token.
- [`read_operator_token_env`](#read_operator_token_env) - Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty.
- [`validate_operator_token`](#validate_operator_token) - Reject tokens that can break URL fragments, HTTP headers, or shell pastes.

---

## bookclerk_config::operator_auth::ResolveOperatorTokenEnv

*Enum*

How an operator token was resolved from env (config-layer only).

**Variants:**
- `Env` - Value from `BOOKCLERK_OPERATOR_TOKEN`.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ResolveOperatorTokenEnv`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ResolveOperatorTokenEnv) -> bool`



## bookclerk_config::operator_auth::generate_operator_token

*Function*

Generate a 32-byte hex operator token.

# Errors

Returns an error when the operation fails.

```rust
fn generate_operator_token() -> crate::error::Result<String>
```



## bookclerk_config::operator_auth::read_operator_token_env

*Function*

Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty.

# Errors

Returns an error when the operation fails.

```rust
fn read_operator_token_env() -> crate::error::Result<Option<(String, ResolveOperatorTokenEnv)>>
```



## bookclerk_config::operator_auth::validate_operator_token

*Function*

Reject tokens that can break URL fragments, HTTP headers, or shell pastes.

Generated tokens are hex. Env overrides must stay printable single-line and
free of whitespace / control characters so they cannot inject into
`Authorization` headers.

# Errors

Returns an error when the operation fails.

```rust
fn validate_operator_token(token: &str, source: &str) -> crate::error::Result<String>
```



