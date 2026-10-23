# bookclerk_library::password

Argon2id password hashing for first-party local login.

| Item | Kind | Summary |
| --- | --- | --- |
| [`hash_password`](#hash_password) | function | Hash a plaintext password with Argon2id (PHC string). |
| [`verify_password`](#verify_password) | function | Verify a plaintext password against a stored PHC hash. |

## Functions

### hash_password

```rust
pub fn hash_password(password: &str) -> Result<String>
```

Hash a plaintext password with Argon2id (PHC string).

#### Arguments

* `password` - Plaintext password; never logged or stored.

#### Returns

PHC-formatted Argon2id hash string for durable storage.

#### Errors

Returns [`LibraryError::Other`](index.md#libraryerror) when hashing fails.

### verify_password

```rust
pub fn verify_password(password: &str, password_hash: &str) -> Result<bool>
```

Verify a plaintext password against a stored PHC hash.

#### Arguments

* `password` - Candidate plaintext password.
* `password_hash` - Stored Argon2id PHC string from [`hash_password`](#hash_password).

#### Returns

`Ok(true)` on match, `Ok(false)` on mismatch.

#### Errors

Returns [`LibraryError::Other`](index.md#libraryerror) when the stored hash cannot be parsed.
