**bookclerk_library > token_hash**

# Module: token_hash

## Contents

**Functions**

- [`derive_claim_password_fingerprint`](#derive_claim_password_fingerprint) - Stable, non-secret fingerprint of an invite/reset password for idempotency.
- [`derive_claim_session_token`](#derive_claim_session_token) - Derive the portal session token for a claim ticket plus browser nonce.
- [`hash_token`](#hash_token) - SHA-256 hex digest of a raw ticket/session token for durable storage.
- [`parse_claim_redeem_nonce`](#parse_claim_redeem_nonce) - Accept a browser-generated claim-redeem nonce.

**Constants**

- [`CLAIM_REDEEM_NONCE_HEX_LEN`](#claim_redeem_nonce_hex_len) - Hex length of a browser-generated claim-redeem nonce (32 bytes).

---

## bookclerk_library::token_hash::CLAIM_REDEEM_NONCE_HEX_LEN

*Constant*: `usize`

Hex length of a browser-generated claim-redeem nonce (32 bytes).



## bookclerk_library::token_hash::derive_claim_password_fingerprint

*Function*

Stable, non-secret fingerprint of an invite/reset password for idempotency.

Argon2id storage hashes use a fresh salt on every POST, so they cannot be
the atomic request hash. This value is
`HMAC-SHA256(subkey, nonce || 0x00 || password)` where
`subkey = HMAC-SHA256(DEK, "bookclerk-claim-password-subkey-v1")`.
The randomized Argon2id hash is still what gets stored on the user row.

```rust
fn derive_claim_password_fingerprint(dek: &crate::master_key::MasterKey, nonce: &str, password: &str) -> String
```



## bookclerk_library::token_hash::derive_claim_session_token

*Function*

Derive the portal session token for a claim ticket plus browser nonce.

The raw session is
`HMAC-SHA256(DEK, "bookclerk-claim-session-v1:" || ticket || 0x00 || nonce)`
hex-encoded (64 characters). The same ticket, nonce, and process DEK always
yield the same session, so a browser retry after a lost atomic reply
reuses `redeemClaimTicket:{token}:{session}`. A different nonce (another
browser, or the magic link alone) cannot replay that receipt. The plaintext
bearer is never written to a receipt; only [`hash_token`](token_hash.md#bookclerk_librarytoken_hashhash_token) of this value is
stored.

# Arguments

* `dek` - Process data-encryption key from [`crate::require_master_key`](master_key.md#bookclerk_librarymaster_keyrequire_master_key).
* `raw_ticket` - Plaintext claim ticket from the SPA / share URL.
* `nonce` - Validated 64-hex redeem nonce from [`parse_claim_redeem_nonce`](token_hash.md#bookclerk_librarytoken_hashparse_claim_redeem_nonce).

```rust
fn derive_claim_session_token(dek: &crate::master_key::MasterKey, raw_ticket: &str, nonce: &str) -> String
```



## bookclerk_library::token_hash::hash_token

*Function*

SHA-256 hex digest of a raw ticket/session token for durable storage.

# Arguments

* `raw` - Opaque plaintext token; callers must not persist this value.

# Returns

Lowercase hex SHA-256 digest (64 characters).

```rust
fn hash_token(raw: &str) -> String
```



## bookclerk_library::token_hash::parse_claim_redeem_nonce

*Function*

Accept a browser-generated claim-redeem nonce.

The SPA persists a 32-byte random value as 64 hex characters across HTTP
retries (`sessionStorage`). Possession of the used magic-link URL without
this nonce cannot recover the committed session.

# Errors

Returns an error when the value is missing, the wrong length, or not hex.

```rust
fn parse_claim_redeem_nonce(raw: &str) -> crate::error::Result<&str>
```



