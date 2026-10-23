**bookclerk_library > password**

# Module: password

## Contents

**Functions**

- [`hash_password`](#hash_password) - Hash a plaintext password with Argon2id (PHC string).
- [`verify_password`](#verify_password) - Verify a plaintext password against a stored PHC hash.

---

## bookclerk_library::password::hash_password

*Function*

Hash a plaintext password with Argon2id (PHC string).

# Arguments

* `password` - Plaintext password; never logged or stored.

# Returns

PHC-formatted Argon2id hash string for durable storage.

# Errors

Returns `LibraryError::Other` when hashing fails.

```rust
fn hash_password(password: &str) -> crate::error::Result<String>
```



## bookclerk_library::password::verify_password

*Function*

Verify a plaintext password against a stored PHC hash.

# Arguments

* `password` - Candidate plaintext password.
* `password_hash` - Stored Argon2id PHC string from [`hash_password`](password.md#bookclerk_librarypasswordhash_password).

# Returns

`Ok(true)` on match, `Ok(false)` on mismatch.

# Errors

Returns `LibraryError::Other` when the stored hash cannot be parsed.

```rust
fn verify_password(password: &str, password_hash: &str) -> crate::error::Result<bool>
```



