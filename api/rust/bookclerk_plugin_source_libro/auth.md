**bookclerk_plugin_source_libro > auth**

# Module: auth

## Contents

**Structs**

- [`LibroAuthFile`](#libroauthfile) - Credential envelope for one Libro.fm account.

---

## bookclerk_plugin_source_libro::auth::LibroAuthFile

*Struct*

Credential envelope for one Libro.fm account.

Persisted as JSON in `encrypted_secrets` (see [`crate::db`](db.md)); never written
to disk under `Accounts/`.

**Fields:**
- `access_token: String` - Bearer token from Libro.fm OAuth (`access_token`).
- `token_type: String` - OAuth token type (almost always `Bearer`).
- `expires_at: Option<chrono::DateTime<chrono::Utc>>` - Absolute expiry when known (RFC3339 when serialized).
- `email: String` - Sign-in email used as fallback account id.
- `user_id: Option<String>` - Libro.fm user id when the token response includes one.
- `marketplace: String` - Marketplace / locale hint (not part of the mobile API; stored for Bookclerk).
- `label: Option<String>` - Optional operator-facing account label in Accounts UI.

**Methods:**

- `fn account_id(self: &Self) -> &str` - Stable account id for library rows: prefer user id, else email.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> LibroAuthFile`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



