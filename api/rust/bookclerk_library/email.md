**bookclerk_library > email**

# Module: email

## Contents

**Functions**

- [`gravatar_hash`](#gravatar_hash) - SHA-256 hex digest of a trimmed, lowercased email for Gravatar URLs.
- [`is_valid_user_email`](#is_valid_user_email) - True when `value` is a syntactically valid email with a dotted domain.
- [`normalize_user_email`](#normalize_user_email) - Trim, lowercase, and validate a contact email; empty becomes `None`.

---

## bookclerk_library::email::gravatar_hash

*Function*

SHA-256 hex digest of a trimmed, lowercased email for Gravatar URLs.

Gravatar matches on this digest; callers should pass a stored/normalized
address. Empty or whitespace-only input still hashes (callers should skip).

```rust
fn gravatar_hash(email: &str) -> String
```



## bookclerk_library::email::is_valid_user_email

*Function*

True when `value` is a syntactically valid email with a dotted domain.

```rust
fn is_valid_user_email(value: &str) -> bool
```



## bookclerk_library::email::normalize_user_email

*Function*

Trim, lowercase, and validate a contact email; empty becomes `None`.

# Errors

Returns `LibraryError::InvalidEmail` when the value is non-empty but not a
valid address (including a domain without a dot, such as `user@gmail`).

```rust
fn normalize_user_email(email: Option<&str>) -> crate::error::Result<Option<String>>
```



