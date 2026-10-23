# bookclerk_library::email

Contact-email normalization and library-backed validation.

| Item | Kind | Summary |
| --- | --- | --- |
| [`normalize_user_email`](#normalize_user_email) | function | Trim, lowercase, and validate a contact email; empty becomes `None`. |
| [`is_valid_user_email`](#is_valid_user_email) | function | True when `value` is a syntactically valid email with a dotted domain. |
| [`gravatar_hash`](#gravatar_hash) | function | SHA-256 hex digest of a trimmed, lowercased email for Gravatar URLs. |

## Functions

### normalize_user_email

```rust
pub fn normalize_user_email(email: Option<&str>) -> Result<Option<String>>
```

Trim, lowercase, and validate a contact email; empty becomes `None`.

#### Errors

Returns [`LibraryError::InvalidEmail`](index.md#libraryerror) when the value is non-empty but not a
valid address (including a domain without a dot, such as `user@gmail`).

### is_valid_user_email

```rust
pub fn is_valid_user_email(value: &str) -> bool
```

True when `value` is a syntactically valid email with a dotted domain.

### gravatar_hash

```rust
pub fn gravatar_hash(email: &str) -> String
```

SHA-256 hex digest of a trimmed, lowercased email for Gravatar URLs.

Gravatar matches on this digest; callers should pass a stored/normalized
address. Empty or whitespace-only input still hashes (callers should skip).
